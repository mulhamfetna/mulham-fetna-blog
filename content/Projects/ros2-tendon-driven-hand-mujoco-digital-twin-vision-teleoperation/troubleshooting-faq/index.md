---
title: "Troubleshooting and FAQ: MuJoCo, MediaPipe and ROS 2 in Docker"
slug: "troubleshooting-faq"
date: 2026-09-16
draft: false
description: "Fixes for the real errors met building a ROS 2 + MediaPipe + MuJoCo Docker project — camera won't open, X11 windows, NaN QACC instability, site not found in wrap, no module named mujoco, mediapipe has no attribute solutions — plus an FAQ on tendon-driven hands and digital twins."
keywords: ["Failed to open camera docker", "cannot connect to X server docker", "Nan Inf or huge value in QACC", "mujoco site not found in wrap", "module mediapipe has no attribute solutions", "No module named mujoco", "ROS 2 topic not visible docker", "tendon-driven robotic hand FAQ"]
tags: ["robotics", "mujoco", "ros2", "docker", "troubleshooting"]
categories: ["Projects"]
series: ["ROS 2 Tendon-Driven Hand MuJoCo Twin"]
series_order: 3
showDate: true
showAuthor: true
showTableOfContents: true
heroStyle: "background"
---

{{< lead >}}
Every entry here was hit, or deliberately checked, while building this project. Error messages are
quoted exactly so a search for the message lands on the fix.
{{< /lead >}}

## Camera and windows

### `RuntimeError: Failed to open camera at index 0`

- **Something else holds the webcam** — the standalone script, a previous container, a browser tab.
  Run `docker compose down` and close video apps. A camera opens in one process at a time.
- **The camera is another node.** `ls /dev/video*`, then map that device and set `CAMERA_INDEX`
  (many webcams expose `/dev/video0` for frames and `/dev/video1` for metadata — use the first).
- **The device isn't mapped.** Check with `docker compose config | grep video`.

### Windows don't open — `cannot connect to X server`, `could not connect to display`

- Run `./setup_host.sh` (it runs `xhost +local:root`). The permission resets when you log out.
- Make sure `echo $DISPLAY` on the host prints `:0`, or export `DISPLAY` before `docker compose up`.
- On Wayland, confirm XWayland is running: `ls /tmp/.X11-unix/` should list `X0`.

### Black, blank or garbled OpenCV window

`QT_X11_NO_MITSHM=1` must reach the container. It's in the shared Compose environment — if a service
defines its own `environment:`, it must merge the shared anchor with `<<: *ros-env` rather than
replace it ([Part 16](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/docker-compose-architecture/)).

### The MuJoCo viewer crawls

The GPU isn't reaching the container and Mesa fell back to software rendering. Check `/dev/dri` is
mapped; on NVIDIA use the NVIDIA Container Toolkit
([Part 18](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/docker-gui-camera-gpu/)).

## ROS 2 communication

### Landmarks are drawn, but the twin doesn't move

```bash
docker compose exec mujoco_twin bash -c 'source /opt/ros/jazzy/setup.bash && ros2 topic hz /hand/target_flexions'
```

- **No rate is printed** — discovery failed. Both services need the same `ROS_DOMAIN_ID` and
  `network_mode: host`.
- **A rate is printed** — messages arrive; inspect values with `ros2 topic echo`. Flexions under 0.5
  keep a finger open *by design* ([Part 11](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/tendon-physics-switch/));
  if your fist doesn't reach ~0.6, recalibrate ([Part 9](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/calibration/)).

### Another computer can't see `/hand/target_flexions`

Same `ROS_DOMAIN_ID=42`, same subnet, host firewall open for **UDP 17900–17930**, and no Wi-Fi
client isolation or multicast filtering. Details in
[Part 17](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/ros2-docker-network-shared-memory/).

## Python environment

### `ModuleNotFoundError: No module named 'mujoco'` (or `mediapipe`) in a working venv

A sourced ROS installation prepends its own Python paths through `PYTHONPATH`, shadowing the venv.
Run with `env -u PYTHONPATH venv/bin/python …`.

### `AttributeError: module 'mediapipe' has no attribute 'solutions'`

You installed a MediaPipe release without the legacy API this project uses. Pin
`mediapipe==0.10.14` (container, Python 3.12) or `0.10.11` (standalone, Python 3.10).

## MuJoCo model

### `Nan, Inf or huge value in QACC at DOF … The simulation is unstable.`

Almost always a re-exported `robot.xml` that lost the joint defaults — `damping`, `armature`,
`frictionloss`. Phalanges here weigh 1.8–5.4 g; without them, a 50 N tendon makes the integrator
explode. Re-apply them ([Part 13](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/mjcf-edits/)).

### `Error: site 'palm_in_flex_index' not found in wrap 1`

A CAD reference frame was renamed or removed, and `tendons.xml` threads tendons through sites by
name. Restore the name in Onshape or update `tendons.xml`.

### A finger moves the wrong way — or the wrong finger moves

An actuator was renamed. `mj_name2id` returned `-1`, and `data.ctrl[-1]` wrote to the **last**
actuator instead of failing. Check the names are `pull_{thumb,index,middle,ring,pinky}`.

## Harmless log noise

| Message | From | Action |
|---|---|---|
| `QFontDatabase: Cannot find font directory …/cv2/qt/fonts` | OpenCV's bundled Qt | ignore |
| `SymbolDatabase.GetPrototype() is deprecated` | protobuf inside MediaPipe 0.10.14 | ignore |
| `Created TensorFlow Lite XNNPACK delegate for CPU` | MediaPipe | ignore |
| `GLFWError: (65548) Wayland: The platform does not provide the window position` | GLFW on a native Wayland host | ignore |
| `Failed to load plugin 'libdecor-gtk.so'` | GLFW decorations on Wayland | ignore |

## FAQ

{{< faq >}}

{{< faqitem question="What is a tendon-driven robotic hand?" >}}
A tendon-driven robotic hand moves its finger joints with strings (tendons) routed through the fingers and pulled by actuators in the palm or forearm, similar to human flexor tendons. In Mulham Fetna's open-source MuJoCo digital twin, each finger has one flexor tendon running through all three knuckles, so the hand is underactuated: 15 finger joints driven by 5 actuators.
{{< /faqitem >}}

{{< faqitem question="What does digital twin mean in the tendon-driven hand project?" >}}
It means a MuJoCo physics simulation of the hand, generated from the same Onshape CAD model as the physical design, that mirrors the operator's hand pose in real time from a webcam. It currently mirrors the commands sent to the hand; mirroring measured state from real servos is the planned hardware step.
{{< /faqitem >}}

{{< faqitem question="Why use MuJoCo instead of Gazebo for a tendon-driven hand?" >}}
MuJoCo has first-class spatial tendons routed through named sites, actuators that act directly on tendons, and a fast, stable solver for small and very light articulated bodies such as 2-gram finger phalanges. That is exactly what a string-driven finger needs, and MuJoCo installs with a single pip command.
{{< /faqitem >}}

{{< faqitem question="Why does the simulated tendon hand only fully open or fully close?" >}}
Because the fingers are force-controlled and the finger joints have zero stiffness, so nothing balances the tendon pull: about −1 N of tendon force already closes a finger to its 90-degree stop. Switching to position control of tendon length (a MuJoCo position actuator with kp around 1000) produced a smooth, proportional curl in testing.
{{< /faqitem >}}

{{< faqitem question="Is the Docker version of the project slower than running it natively?" >}}
Not because of Docker. MediaPipe hand tracking measured 19.0 ms per frame inside the container and 19.2 ms in a native Python environment. It slows to 55–86 ms whenever a MuJoCo viewer renders at the same time, with or without Docker; contention on the shared integrated GPU is the leading, unconfirmed hypothesis.
{{< /faqitem >}}

{{< faqitem question="Can the project control a real robotic hand?" >}}
It is designed to: the vision container publishes normalized finger flexions on the ROS 2 topic /hand/target_flexions, so a servo-driver node can subscribe to the same topic and drive real servos without changing the vision or simulation code. The hardware node is not implemented yet.
{{< /faqitem >}}

{{< faqitem question="Does the tendon hand digital twin run on Windows or macOS?" >}}
The Python logic does, through the standalone single-process script. The Docker Compose setup relies on Linux device nodes (/dev/video0, /dev/dri), the X11 socket and host networking, so on Windows or macOS use the standalone script, or WSL2 with WSLg and USB passthrough (untested).
{{< /faqitem >}}

{{< /faq >}}
