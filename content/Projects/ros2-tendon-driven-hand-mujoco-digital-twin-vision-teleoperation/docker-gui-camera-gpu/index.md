---
title: "GUI, webcam and GPU passthrough for ROS 2 and MuJoCo containers"
slug: "docker-gui-camera-gpu"
date: 2026-09-16
draft: false
description: "Run OpenCV and MuJoCo GUI windows from Docker containers on Linux (including Wayland via XWayland): X11 socket passthrough, xhost, QT_X11_NO_MITSHM, /dev/dri GPU rendering, NVIDIA notes, MUJOCO_GL=glfw, /dev/video0 webcam access with V4L2, and the security cost of privileged containers."
keywords: ["docker GUI X11 Wayland", "docker webcam /dev/video0", "docker GPU /dev/dri Mesa", "MUJOCO_GL glfw docker", "QT_X11_NO_MITSHM", "xhost +local:root", "docker privileged security", "OpenCV imshow docker"]
tags: ["docker", "linux", "ros2", "mujoco"]
categories: ["Projects"]
series: ["ROS 2 Tendon-Driven Hand MuJoCo Twin"]
series_order: 18
showDate: true
showAuthor: true
showTableOfContents: true
heroStyle: "background"
---

{{< lead >}}
Containers are headless by design. This project needs two windows, a webcam and a GPU — so the
Compose file spends most of its lines punching carefully chosen holes back through the isolation.
{{< /lead >}}

![Both GUI windows from two containers running together on one display](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/hero_teleoperation.gif "Two containers, two windows, one X display: the Qt/OpenCV tracker (vision_tracker) and the GLFW MuJoCo viewer (mujoco_twin), recorded on Wayland through XWayland.")

## Windows: pass the X11 socket through

```yaml
environment:
  DISPLAY: ${DISPLAY:-:0}
  QT_X11_NO_MITSHM: 1
volumes:
  - /tmp/.X11-unix:/tmp/.X11-unix:rw
```

| Piece | What it does |
|---|---|
| `/tmp/.X11-unix` mount | the X server listens on a Unix socket here (`X0` for `:0`); mounting it gives the container a line to it |
| `DISPLAY` | tells X clients (GLFW, Qt) which display to use; defaults to `:0` |
| `QT_X11_NO_MITSHM=1` | stops Qt — OpenCV's `imshow` backend — from using MIT-SHM, which fails across containers and gives blank or garbled windows |
| `xhost +local:root` (in `setup_host.sh`) | the X server refuses untrusted clients; this admits local root, which is who the containers run as |

### On Wayland

Wayland sessions (KDE Plasma, GNOME) still run **XWayland** on `:0`, and both windows open through it —
the recording above was made exactly that way. Running MuJoCo *natively* on a Wayland host, GLFW may
warn `Wayland: The platform does not provide the window position`; it's harmless.

## GPU: `/dev/dri`

```yaml
devices:
  - /dev/dri:/dev/dri
```

`/dev/dri` holds the Direct Rendering Infrastructure nodes (`card*`, `renderD*`). With them, Mesa in the
container renders on the real GPU; without them it falls back to CPU software rendering and the viewer
crawls.

{{< tabs >}}
{{< tab label="Intel / AMD" >}}
`/dev/dri` is all you need — Mesa drives the GPU from inside the container. This project was developed on
Intel UHD Graphics (Comet Lake).
{{< /tab >}}
{{< tab label="NVIDIA" >}}
The proprietary driver isn't reached through `/dev/dri`. Install the NVIDIA Container Toolkit, then add a
GPU reservation (`deploy.resources.reservations.devices` with `capabilities: [gpu]`, or
`runtime: nvidia`) and `NVIDIA_DRIVER_CAPABILITIES=all` to the services.
{{< /tab >}}
{{< /tabs >}}

`MUJOCO_GL=glfw` selects MuJoCo's windowed backend, which the interactive viewer needs; `egl` and
`osmesa` are for headless offscreen rendering.

MediaPipe gets `/dev/dri` too. On the development host, its log showed that even the CPU hand-tracking
graph opens an **EGL context on the GPU** at startup
(`gl_context_egl.cc … Successfully initialized EGL … Mesa Intel(R) UHD Graphics`) — which may matter for
performance ([Part 19](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/latency-benchmark/)).

## Webcam: `/dev/video0`

```yaml
devices:
  - /dev/video0:/dev/video0
environment:
  CAMERA_INDEX: 0
```

- OpenCV's **V4L2** backend opens `/dev/video<CAMERA_INDEX>`. Verified inside the container: V4L2,
  640×480, 30 fps, YUYV.
- Many webcams expose **two** nodes — `/dev/video0` for frames, `/dev/video1` for metadata. Use the first.
- Another camera: map it and match the index, e.g. `/dev/video2` with `CAMERA_INDEX: 2`.
- `v4l-utils` is in the image: `docker compose exec vision_tracker v4l2-ctl --list-formats-ext`.
- **One process per camera.** Stop the standalone script and close browser tabs using the webcam first.

## `setup_host.sh`

```bash
./setup_host.sh
```

It runs `xhost +local:root` (asking you to install `x11-xserver-utils` if `xhost` is missing), then checks
that `/dev/video0` and `/dev/dri` exist. `xhost` grants reset at logout — run it **once per login**.

## `privileged: true` — and what it really means

`privileged` hands the containers **every** host device and almost every kernel capability. It's there for
convenience: hot-plugged cameras and GPU nodes just work. Combined with host network, IPC and PID, and
`xhost +local:root`, **these containers are effectively not sandboxed from the host**. Fine on a trusted
workstation running your own code; not fine anywhere else.

{{< alert icon="shield" >}}
**Hardening, in order of effort:**
1. Drop `privileged: true` — the explicit `devices:` already grant camera and GPU; add
   `group_add: ["video", "render"]` if permissions complain.
2. Replace `xhost +local:root` with a per-container Xauthority cookie, or
   `xhost +SI:localuser:$(id -un)` and run as your user.
3. Run as non-root (`user: "${UID}:${GID}"`) with the **same** UID in both services, so Fast DDS shared
   memory keeps working.
4. Narrow the `ipc`/`pid` sharing ([Part 17](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/ros2-docker-network-shared-memory/)).
{{< /alert >}}
