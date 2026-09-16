---
title: "What limits the frame rate: MediaPipe, Docker and a MuJoCo viewer on one laptop"
slug: "latency-benchmark"
date: 2026-09-16
draft: false
description: "A measured latency benchmark of MediaPipe hand tracking next to a MuJoCo viewer: 19 ms per frame in Docker and natively, 55–86 ms whenever a viewer renders concurrently. What was ruled out (Docker, ROS 2, CPU load, loop rate), the GPU-contention hypothesis, next experiments, and a reproducible script."
keywords: ["MediaPipe latency benchmark", "MediaPipe slow with OpenGL", "Docker performance overhead Python", "MuJoCo viewer performance", "ROS 2 topic hz", "integrated GPU contention", "profiling computer vision pipeline"]
tags: ["performance", "computer-vision", "mujoco", "docker", "benchmark"]
categories: ["Projects"]
series: ["ROS 2 Tendon-Driven Hand MuJoCo Twin"]
series_order: 19
showDate: true
showAuthor: true
showTableOfContents: true
heroStyle: "background"
---

{{< lead >}}
The single-script version felt smoother than the containers. The obvious suspect was Docker. The
measurements say otherwise — and point at something more interesting: two programs sharing one
integrated GPU.
{{< /lead >}}

## Setup

Laptop with an Intel Comet Lake CPU (12 threads) and Intel UHD Graphics; Linux on Wayland with
XWayland; USB webcam at 640×480, 30 fps, YUYV. Each row: mean of 150 frames after a 10-frame warm-up,
no hand in view. Measured 2026-09-16.

## The result

{{< chart >}}
type: 'bar',
data: {
  labels: ['Vision alone (container)', 'Vision alone (host venv)', '+ 1-core CPU burner', '+ MuJoCo viewer (host, no Docker)', '+ mujoco_twin (loop capped 30 Hz)', '+ mujoco_twin (as shipped)'],
  datasets: [{
    label: 'MediaPipe time per frame (ms)',
    data: [19.0, 19.2, 23.5, 54.9, 63.7, 85.6],
    backgroundColor: ['rgba(42,120,214,0.75)','rgba(42,120,214,0.75)','rgba(42,120,214,0.75)','rgba(235,104,52,0.75)','rgba(235,104,52,0.75)','rgba(235,104,52,0.75)']
  }]
},
options: {
  indexAxis: 'y',
  plugins: { legend: { display: false }, title: { display: true, text: 'MediaPipe inference time by what else is running (30 fps budget = 33 ms)' } },
  scales: { x: { min: 0, max: 100, title: { display: true, text: 'ms per frame' } } }
}
{{< /chart >}}

| Condition | Camera read | MediaPipe | `imshow` | Loop |
|---|---|---|---|---|
| Vision alone, **container** | 10.0 ms | **19.0 ms** | 4.5 ms | **29.8 Hz** |
| Vision alone, **host venv** (MediaPipe 0.10.11) | 10.6 ms | **19.2 ms** | 3.9 ms | **29.7 Hz** |
| Container vision + **1-core CPU burner** | 8.2 ms | 23.5 ms | 3.9 ms | 28.1 Hz |
| Host vision + **plain MuJoCo viewer on host** (no Docker, no ROS) | 1.6 ms | **54.9 ms** | 5.9 ms | 16.0 Hz |
| Container vision + **`mujoco_twin` capped at 30 Hz** | 1.7 ms | **63.7 ms** | 8.1 ms | 13.6 Hz |
| Container vision + **`mujoco_twin` as shipped** | 1.9 ms | **85.6 ms** | 10.2 ms | **10.2 Hz** |

End to end, `ros2 topic hz /hand/target_flexions` read **30.0 Hz** with only `vision_tracker` running and
**10.7 Hz** once `mujoco_twin` started.

*The camera-read time drops when the loop slows, because a frame is already waiting in the driver buffer.*

## What it rules out

| Hypothesis | Test | Verdict |
|---|---|---|
| Docker overhead | same benchmark, container vs host venv | ✘ — 19.0 vs 19.2 ms |
| MediaPipe version (0.10.14 vs 0.10.11) | same comparison | ✘ |
| ROS 2 / DDS | a plain viewer with no ROS still slows MediaPipe | ✘ as the cause |
| General CPU load | busy loop on one core | ✘ — +4 ms |
| The twin's unthrottled loop | cap it at 30 Hz | ✘ — still 64 ms |
| **Any concurrently rendering MuJoCo viewer** | all three viewer rows | ✔ **3–4.5× slower, every time** |

## The leading hypothesis: one iGPU, two OpenGL programs

- MediaPipe's own log shows it creating an **EGL context on the Intel iGPU** at startup — even for the CPU graph.
- The MuJoCo viewer renders on the **same** iGPU through GLFW and XWayland.
- CPU load alone doesn't reproduce the slowdown; a concurrent viewer does, even at 30 Hz and without Docker.
- The standalone script runs both in **one** process and syncs the viewer only once per camera frame.

{{< alert icon="circle-question" >}}
**Not confirmed.** The investigation stopped here because the pipeline worked well enough. Treat GPU
contention as the best current explanation, not a finding.
{{< /alert >}}

Next experiments, one variable at a time:

1. Start the vision container **without** `/dev/dri` (and without `privileged`, which exposes it anyway)
   so MediaPipe can't open its EGL context on the iGPU. Does the viewer still slow it?
2. Run `mujoco_twin` headless with `MUJOCO_GL=egl` and no window. If MediaPipe recovers, rendering is the trigger.
3. Render MuJoCo on a different GPU — discrete or NVIDIA — and repeat.
4. Instrument `standalone/main.py` the same way for a like-for-like single-process baseline.

## Cheap wins regardless of the cause

| Change | Expected effect |
|---|---|
| Throttle `viewer.sync()` to ~60 Hz while physics keeps stepping at 500 Hz | fewer render submissions on the shared GPU, less CPU |
| Drop or decimate `cv2.imshow` in headless deployments | saves 4–10 ms per frame |
| Capture at 320×240 | MediaPipe crops the hand anyway |
| Grab frames on a thread; process only the newest | bounded latency when inference is slower than the camera |

## Reproduce it

```python
import time, cv2, mediapipe as mp
cap = cv2.VideoCapture(0)
hands = mp.solutions.hands.Hands(static_image_mode=False, max_num_hands=1,
                                 min_detection_confidence=0.5, min_tracking_confidence=0.5)
for _ in range(10): cap.read()                                   # warm-up
N = 150; tr = tp = ts = 0.0; t0 = time.perf_counter()
for i in range(N):
    a = time.perf_counter(); ok, f = cap.read(); b = time.perf_counter()
    hands.process(cv2.cvtColor(f, cv2.COLOR_BGR2RGB)); c = time.perf_counter()
    cv2.imshow("bench", f); cv2.waitKey(1); d = time.perf_counter()
    tr += b - a; tp += c - b; ts += d - c
T = time.perf_counter() - t0
print(f"loop {N/T:.1f} Hz | read {1000*tr/N:.1f} ms | mediapipe {1000*tp/N:.1f} ms | imshow {1000*ts/N:.1f} ms")
```

```bash
env -u PYTHONPATH venv/bin/python bench.py                                    # host
docker compose run --rm --no-deps -v "$PWD":/bench:ro --entrypoint bash \
  vision_tracker -c 'python3 /bench/bench.py'                                 # container
docker compose up -d mujoco_twin                                              # then repeat with the twin running
```

Stop the full stack first — a camera opens in one process at a time.
