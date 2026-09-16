---
title: "Calibrating hand tracking to your own hand — with real data from a live session"
slug: "calibration"
date: 2026-09-16
draft: false
description: "A step-by-step procedure to calibrate MediaPipe finger-flexion thresholds for robot teleoperation, plus real per-finger flexions recovered from MuJoCo's live motor forces: an open hand at 0.02–0.06, but a fist that only reaches 0.61–0.76."
keywords: ["hand tracking calibration", "MediaPipe calibration", "finger flexion calibration", "teleoperation calibration procedure", "per finger calibration", "MuJoCo control panel", "robot hand calibration data"]
tags: ["computer-vision", "robotics", "mediapipe", "tutorial"]
categories: ["Projects"]
series: ["ROS 2 Tendon-Driven Hand MuJoCo Twin"]
series_order: 9
showDate: true
showAuthor: true
showTableOfContents: true
heroStyle: "background"
---

{{< katex >}}

{{< lead >}}
The shipped thresholds were measured on one hand with one webcam. Here is how to measure yours — and
what a live recording revealed about how far off "good enough" can be while the twin still looks
perfect.
{{< /lead >}}

## Symptoms that call for calibration

| What you see in the published flexions | Cause | Change |
|---|---|---|
| A fist, but flexion stays below 1.0 | your fist angle is **above** the curled limit | **raise** `*_CURLED_ANGLE` to your fist reading |
| Flexion hits 1.0 with the hand half closed | curled limit too high | **lower** `*_CURLED_ANGLE` |
| Flexion above 0 with a relaxed open hand | your open angle is **below** the straight limit | **lower** `*_STRAIGHT_ANGLE` to your open reading |
| The thumb flickers | thumb window too narrow for your jitter | widen it ([Part 8](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/thumb-thresholds/)) |

{{< alert icon="lightbulb" >}}
**Calibrate against the numbers, not the render.** With the current force-control tuning, the
simulated finger snaps shut past flexion ≈ 0.51 ([Part 11](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/tendon-physics-switch/)),
so most calibration errors are invisible in the viewer. Watch `ros2 topic echo /hand/target_flexions`.
{{< /alert >}}

## Real data: what a live session produced

MuJoCo's viewer has a **Control** panel showing the live force of every motor. Since the force is
\(F = 50 - 100 \cdot \text{flexion}\), every recorded frame gives back the exact flexion the tracker
published: **flexion = (50 − F) / 100**.

{{< carousel images="{live_open_hand.jpg,live_fist.jpg,live_ok_sign.jpg}" aspectRatio="16-5" interval="3500" captions="{live_open_hand.jpg:Open hand,live_fist.jpg:Fist,live_ok_sign.jpg:OK sign}" >}}

| Pose | pinky | ring | middle | index | thumb |
|---|---|---|---|---|---|
| Open hand | +43.9 N → **0.06** | +47.0 N → **0.03** | +47.6 N → **0.02** | +46.5 N → **0.04** | +30.2 N → **0.20** |
| Peace sign | −21.0 N → **0.71** | −20.9 N → **0.71** | +39.1 N → **0.11** | +44.4 N → **0.06** | +43.8 N → **0.06** |
| Fist | −26.1 N → **0.76** | −26.2 N → **0.76** | −17.2 N → **0.67** | −17.7 N → **0.68** | −11.4 N → **0.61** |
| OK sign | +48.1 N → **0.02** | +45.8 N → **0.04** | +45.8 N → **0.04** | −9.7 N → **0.60** | −32.9 N → **0.83** |
| Thumb curled | +47.5 N → **0.03** | +47.3 N → **0.03** | +47.5 N → **0.03** | +45.2 N → **0.05** | −19.5 N → **0.69** |

{{< chart >}}
type: 'bar',
data: {
  labels: ['pinky', 'ring', 'middle', 'index', 'thumb'],
  datasets: [
    { label: 'Open hand', data: [0.06, 0.03, 0.02, 0.04, 0.20], backgroundColor: 'rgba(42, 120, 214, 0.75)' },
    { label: 'Fist', data: [0.76, 0.76, 0.67, 0.68, 0.61], backgroundColor: 'rgba(235, 104, 52, 0.75)' }
  ]
},
options: {
  plugins: { title: { display: true, text: 'Published flexion per finger, recovered from live motor forces' } },
  scales: { y: { min: 0, max: 1, title: { display: true, text: 'flexion (ideal: open 0, fist 1)' } } }
}
{{< /chart >}}

Reading it like a calibrator:

- **The open hand is well calibrated** — fingers at 0.02–0.06. The thumb at 0.20 means this
  operator's relaxed thumb sits about 0.12 rad below `THUMB_STRAIGHT_ANGLE`; lowering it by ~0.1
  brings the open thumb to ~0.
- **The fist never reaches 1.0.** Fingers top out at 0.67–0.76, the thumb at 0.61 — the fist angles
  sit above the curled limits. Raising `RAW_CURLED_ANGLE` (and `THUMB_CURLED_ANGLE`) toward the
  measured fist values would use the full range.
- **Middle and index curl less than ring and pinky** in the same fist (0.67 vs 0.76) — an argument
  for per-finger windows.
- **The twin still looked right in every pose**, because anything above ~0.51 already closes a
  finger. These gaps only become visible once actuation is proportional.

## The procedure

{{< timeline >}}

{{< timelineItem icon="code" header="1 · Print the raw angles" subheader="Temporary diagnostic in get_finger_flexions" md="true" >}}
```python
for finger_name, triplets in self.finger_triplets.items():
    angles = [self._calculate_angle(lm[p1], lm[p2], lm[p3]) for p1, p2, p3 in triplets]
    avg_angle = np.mean(angles)
    print(f"{finger_name:>6}: {avg_angle:.2f}", end="  ")   # TEMPORARY
```
Add a bare `print()` after the loop to end the line.
{{< /timelineItem >}}

{{< timelineItem icon="eye" header="2 · Run it" subheader="Standalone for the fastest feedback" md="true" >}}
`env -u PYTHONPATH venv/bin/python standalone/main.py` — or in containers,
`docker compose up vision_tracker` and `docker compose logs -f vision_tracker`.
{{< /timelineItem >}}

{{< timelineItem icon="sun" header="3 · Measure open" subheader="Relaxed, not hyperextended" md="true" >}}
Hold your hand the way you'll hold it while operating. Read the value each finger **settles around**
— typically 3.0–3.1 for fingers and 2.7–3.0 for the thumb. These become `RAW_STRAIGHT_ANGLE` and
`THUMB_STRAIGHT_ANGLE`.
{{< /timelineItem >}}

{{< timelineItem icon="moon" header="4 · Measure closed" subheader="A comfortable full fist, thumb tucked" md="true" >}}
Typically 1.5–1.8 for fingers and 2.1–2.4 for the thumb. These become `RAW_CURLED_ANGLE` and
`THUMB_CURLED_ANGLE`.
{{< /timelineItem >}}

{{< timelineItem icon="pencil" header="5 · Write the constants — in every copy" subheader="Leave a 0.05 rad margin" md="true" >}}
Set the straight limit ~0.05 rad **below** your open reading and the curled limit ~0.05 rad
**above** your fist reading, so relaxed poses reliably clip to exactly 0 and 1. The constants live in
`standalone/main.py`, `vision_tracker/src/vision_tracker_node.py`, and the guard in
`docs/tools/make_figures.py`. For the container, `docker compose restart vision_tracker` — no rebuild.
{{< /timelineItem >}}

{{< timelineItem icon="xmark" header="6 · Remove the print" md="true" >}}
Printing 30 times a second costs real time in the loop.
{{< /timelineItem >}}

{{< /timeline >}}

## Going further: per-finger windows

The data above already argues for it:

```python
CALIBRATION = {                     # (straight, curled) in radians — measure your own
    "thumb":  (2.80, 2.50),
    "index":  (3.10, 2.10),
    "middle": (3.10, 2.10),
    "ring":   (3.10, 1.95),
    "pinky":  (3.10, 1.95),
}
```

*(Values back-computed from the live table above — e.g. a finger fist flexion of 0.67 means an average angle of 3.10 − 0.67 × 1.50 ≈ 2.10 rad. Measure your own rather than copying.)* In
production these belong in a ROS 2 parameter file, so recalibrating never touches source code.
