---
title: "Docker Compose architecture for ROS 2: dependency-only images and mounted code"
slug: "docker-compose-architecture"
date: 2026-09-16
draft: false
description: "A Docker Compose layout for a two-container ROS 2 Jazzy robotics project: per-service build contexts, ros:jazzy images that hold only dependencies, a venv with system site packages, pinned MediaPipe and MuJoCo, the whole repo bind-mounted read-only, and a shared YAML anchor for network, IPC and GUI settings."
keywords: ["ROS 2 Docker Compose", "ros:jazzy Dockerfile", "Docker bind mount development", "docker compose YAML anchor merge", "PEP 668 venv system-site-packages", "robotics Docker project structure", "MuJoCo Docker"]
tags: ["docker", "ros2", "robotics", "devops"]
categories: ["Projects"]
series: ["ROS 2 Tendon-Driven Hand MuJoCo Twin"]
series_order: 16
showDate: true
showAuthor: true
showTableOfContents: true
heroStyle: "background"
---

{{< lead >}}
Both images hold dependencies and nothing else. The code and the robot model are mounted from your
checkout at runtime — so an edit is a five-second restart, not a five-minute rebuild.
{{< /lead >}}

## The big picture

{{< mermaid >}}
flowchart TB
    subgraph HOST["🐧 Linux host"]
        CAMDEV["/dev/video0"]
        GPU["/dev/dri · Intel iGPU"]
        X11["/tmp/.X11-unix<br>XWayland :0"]
        SHM["/dev/shm<br>Fast DDS segments"]
        NET["host network<br>UDP multicast · domain 42"]
        REPO["repository checkout"]
        subgraph VT["🐳 vision_tracker · 2.9 GB image"]
            VN["vision_tracker_node.py<br>mediapipe 0.10.14 · OpenCV"]
        end
        subgraph MT["🐳 mujoco_twin · 1.7 GB image"]
            MN["mujoco_twin_node.py<br>mujoco 3.13.0 · GLFW"]
        end
    end
    CAMDEV --> VN
    GPU --> VN
    GPU --> MN
    X11 <--> VN
    X11 <--> MN
    VN <--> SHM <--> MN
    VN <--> NET <--> MN
    REPO -. "bind mount .:/workspace:ro" .-> VN
    REPO -. "bind mount .:/workspace:ro" .-> MN
{{< /mermaid >}}

## Repository layout

```text
.
├── docker-compose.yml          # both services, shared namespaces
├── setup_host.sh               # xhost + device checks, once per login
├── vision_tracker/
│   ├── Dockerfile              # ros:jazzy + mediapipe==0.10.14
│   ├── .dockerignore           # src/ is mounted, so keep it out of the build context
│   └── src/vision_tracker_node.py
├── mujoco_twin/
│   ├── Dockerfile              # ros:jazzy + mujoco==3.13.0
│   ├── .dockerignore           # src/ and model/ are mounted
│   ├── src/mujoco_twin_node.py
│   └── model/                  # scene.xml → robot.xml (+ tendons.xml), assets/, config.json
├── standalone/main.py          # the same pipeline, one process
└── docs/
```

Each service folder is its own **build context**: editing the vision Dockerfile never invalidates the
twin's image cache, and neither build uploads the 13 MB of meshes it doesn't need.

## Images hold dependencies

Both Dockerfiles follow one pattern:

{{< tabs >}}
{{< tab label="mujoco_twin/Dockerfile" >}}
```dockerfile
FROM ros:jazzy

# OpenGL + GLFW/X11 libs for the MuJoCo passive viewer
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3-pip python3-venv libgl1 libegl1 libglfw3 libx11-6 libxcursor1 \
    libxi6 libxinerama1 libxrandr2 libxkbcommon0 mesa-utils \
    && rm -rf /var/lib/apt/lists/*

RUN python3 -m venv --system-site-packages /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
RUN pip install --no-cache-dir "mujoco==3.13.0"

WORKDIR /root
ENTRYPOINT ["/bin/bash", "-c", "\
  source /opt/ros/jazzy/setup.bash && \
  exec python3 /workspace/mujoco_twin/src/mujoco_twin_node.py"]
```
{{< /tab >}}
{{< tab label="vision_tracker/Dockerfile" >}}
```dockerfile
FROM ros:jazzy

# runtime deps of the OpenCV wheel MediaPipe pulls in (cv2.imshow over X11)
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3-pip python3-venv libgl1 libglib2.0-0 libsm6 libxext6 libxrender1 \
    libxkbcommon-x11-0 libxcb-icccm4 libxcb-image0 libxcb-keysyms1 libxcb-randr0 \
    libxcb-render-util0 libxcb-shape0 libxcb-xinerama0 libxcb-xkb1 v4l-utils \
    && rm -rf /var/lib/apt/lists/*

RUN python3 -m venv --system-site-packages /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
RUN pip install --no-cache-dir "mediapipe==0.10.14"

WORKDIR /root
ENTRYPOINT ["/bin/bash", "-c", "\
  source /opt/ros/jazzy/setup.bash && \
  exec python3 /workspace/vision_tracker/src/vision_tracker_node.py"]
```
{{< /tab >}}
{{< /tabs >}}

| Decision | Why |
|---|---|
| **`FROM ros:jazzy`** | Ubuntu 24.04, Python 3.12, `rclpy`, Fast DDS — no host ROS install |
| **venv with `--system-site-packages`** | Ubuntu 24.04 blocks system `pip install` (PEP 668); the venv takes pip packages while apt-installed `rclpy` dependencies stay importable |
| **`mediapipe==0.10.14`** | later releases removed the `mp.solutions` API |
| **`mujoco==3.13.0`** | the version the standalone pipeline was validated with |
| **`source setup.bash` then `exec python3`** | ROS environment in place; `exec` replaces the shell so `docker stop`'s SIGTERM reaches the node, not a wrapper `bash` |
| **`WORKDIR /root`** | the mount is read-only, and MuJoCo writes `MUJOCO_LOG.TXT` to the working directory |
| **No `COPY` of source** | the code is mounted |

## The code is mounted

```yaml
volumes:
  - .:/workspace:ro
```

- **Edit → restart, no rebuild** — `docker compose restart <service>` takes seconds.
- **One source of truth** — the model simulated is the file in your checkout.
- **A relative path** — the repo works from any clone location.
- **Read-only** — a container can't modify your checkout.

Rebuild only when a Dockerfile or a pinned dependency changes.

## The shared anchor

```yaml
x-ros-common: &ros-common
  network_mode: host
  ipc: host
  pid: host
  environment: &ros-env
    ROS_DOMAIN_ID: 42
    ROS_AUTOMATIC_DISCOVERY_RANGE: SUBNET
    PYTHONUNBUFFERED: 1
    DISPLAY: ${DISPLAY:-:0}
    QT_X11_NO_MITSHM: 1
  volumes:
    - .:/workspace:ro
    - /tmp/.X11-unix:/tmp/.X11-unix:rw
  privileged: true

services:
  vision_tracker:
    <<: *ros-common
    build: { context: ./vision_tracker }
    environment:
      <<: *ros-env
      CAMERA_INDEX: 0
      VISION_RATE_HZ: 30
    devices: [/dev/video0:/dev/video0, /dev/dri:/dev/dri]
  mujoco_twin:
    <<: *ros-common
    build: { context: ./mujoco_twin }
    environment:
      <<: *ros-env
      MUJOCO_GL: glfw
      MUJOCO_SCENE: /workspace/mujoco_twin/model/scene.xml
    devices: [/dev/dri:/dev/dri]
```

{{< alert icon="triangle-exclamation" >}}
**The nested merge matters.** A service's own `environment:` *replaces* the anchor's environment
instead of extending it — unless it merges `<<: *ros-env` too. Forget it and the service silently loses
`ROS_DOMAIN_ID` and `QT_X11_NO_MITSHM`.
{{< /alert >}}

| Setting | Purpose | Deep dive |
|---|---|---|
| `network_mode: host` | DDS discovery between containers and across the LAN | [Part 17](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/ros2-docker-network-shared-memory/) |
| `ipc: host`, `pid: host` | Fast DDS shared-memory transport | [Part 17](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/ros2-docker-network-shared-memory/) |
| `ROS_DOMAIN_ID`, discovery range | which ROS graph, how far discovery reaches | [Part 17](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/ros2-docker-network-shared-memory/) |
| `PYTHONUNBUFFERED` | log lines appear in `docker compose logs` immediately | — |
| `DISPLAY`, X11 socket, `QT_X11_NO_MITSHM`, `/dev/dri` | windows and GPU rendering | [Part 18](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/docker-gui-camera-gpu/) |
| `devices: /dev/video0` | the webcam | [Part 18](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/docker-gui-camera-gpu/) |
| `privileged: true` | development convenience — with a real security cost | [Part 18](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/docker-gui-camera-gpu/) |

## Everyday commands

```bash
./setup_host.sh                            # once per login
docker compose up --build                  # build (first time ~5 min) and start both
docker compose up -d                       # detached
docker compose logs -f vision_tracker      # follow one service
docker compose restart mujoco_twin         # pick up Python/XML edits
docker compose up mujoco_twin              # twin only; drive it with ros2 topic pub
docker compose exec mujoco_twin bash       # a shell inside; then source /opt/ros/jazzy/setup.bash
docker compose down                        # stop and free the camera
```

A healthy start, captured from real runs:

```text
$ docker compose up --build
 Image ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation-mujoco_twin Built
 Image ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation-vision_tracker Built
 Container mujoco_twin Started
 Container vision_tracker Started
mujoco_twin     | [INFO] [1789562507.994533012] [mujoco_twin_node]: MuJoCo twin listening on /hand/target_flexions

$ docker compose exec mujoco_twin bash -c 'source /opt/ros/jazzy/setup.bash && ros2 topic hz /hand/target_flexions'
average rate: 10.863
        min: 0.081s max: 0.112s std dev: 0.00862s window: 30
```
