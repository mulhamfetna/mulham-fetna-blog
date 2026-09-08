---
title: "ROS 2 MediaPipe Robotic Hand — A Real-Time Teleoperation Digital Twin"
description: "A 15-DOF robotic hand driven live by a $20 webcam: MediaPipe hand tracking, explicit dot-product kinematics, a CAD-exported URDF, and four ROS 2 Jazzy containers. Open source under AGPL-3.0 with a Zenodo DOI, plus a six-part engineering series."
keywords: ["ROS 2", "MediaPipe", "robotic hand", "digital twin", "teleoperation", "URDF", "forward kinematics", "hand tracking", "Onshape", "Gazebo", "Docker robotics"]
draft: false
---

{{< github repo="mulhamfetna/ros2-mediapipe-robotic-hand-digital-twin-vision-teleoperation" showThumbnail=true >}}

No gloves. No markers. No depth sensor. A standard RGB webcam watches your hand, and a 15-DOF
robotic hand — designed in CAD, exported to URDF, rendered in ROS 2 — mirrors it in real time.

The interesting part is not that it works. It is that **every joint angle is traceable**. There is
no learned pose-to-pose mapping and no black box between the camera and the mesh: three landmark
coordinates form two vectors, a dot product gives an interior angle, that angle is normalized into
a flexion ratio, and the ratio is linearly interpolated onto the true mechanical limits of the
corresponding joint in the CAD model. Every wrong movement has a findable cause — which is exactly
what you want from a system you intend to attach to real servos.

The code is open under AGPL-3.0 and DOI-archived
([10.5281/zenodo.22658556](https://doi.org/10.5281/zenodo.22658556)), and the
[Onshape assembly is public](https://cad.onshape.com/documents/a2dbb5f16624f10f1aa22f02/w/3eff80c19eddad52bfa92f87/e/4d69727744037003575f4068).

## The pipeline

{{< mermaid >}}
flowchart LR
    A["📷 Webcam<br>/dev/video0"] --> B["BlazePalm<br>palm detector"]
    B --> C["Landmark regressor<br>21 × (x, y, z)"]
    C --> D["Dot-product geometry<br>15 interior angles"]
    D --> E["Normalize → flexion<br>0.0 straight · 1.0 curled"]
    E --> F["Lerp onto the URDF's<br>mechanical limits"]
    F --> G["/joint_states"]
    G --> H["robot_state_publisher<br>→ /tf"]
    H --> I["🖥️ RViz digital twin"]
{{< /mermaid >}}

## The six-part series

Written from the inside — the architecture, the mathematics, and the parts that went wrong.

1. **[A webcam, some vector geometry, and a hand that moves](/projects/ros2-mediapipe-robotic-hand-digital-twin-vision-teleoperation/overview/)** — the whole arc in one read. Start here.
2. **[How MediaPipe sees a hand](/projects/ros2-mediapipe-robotic-hand-digital-twin-vision-teleoperation/mediapipe/)** — two cascaded networks, a faked depth axis, and the tracking loop that lets the detector sleep.
3. **[From camera coordinates to mechanical radians](/projects/ros2-mediapipe-robotic-hand-digital-twin-vision-teleoperation/kinematics/)** — the dot-product engine, the calibration window, and the mapping table that binds vision to CAD.
4. **[Why ROS 2 earns its complexity](/projects/ros2-mediapipe-robotic-hand-digital-twin-vision-teleoperation/ros2/)** — topics, DDS, RViz versus Gazebo, and this project's actual node graph.
5. **[From an Onshape assembly to a robot ROS 2 can reason about](/projects/ros2-mediapipe-robotic-hand-digital-twin-vision-teleoperation/onshape-urdf/)** — five CAD rules, the exporter, and an honest audit of what this export got wrong.
6. **[Containerizing ROS 2 without losing the hardware](/projects/ros2-mediapipe-robotic-hand-digital-twin-vision-teleoperation/docker-gazebo/)** — webcam, GPU and X11 passthrough, and the road to physics.

## What it does not do

Worth stating up front, because it bounds what this demonstrates:

- **No hardware actuation.** The URDF describes geometry and mass, not motors. No transmissions,
  no controllers, no servos.
- **No working physics.** The Gazebo container spawns the model into an empty world, but nothing
  drives it there. The functioning twin is the RViz one, and RViz is a visualizer.
- **No smoothing.** Landmark jitter passes straight through to the joint angles.
- **Forward kinematics only**, joint by joint. No inverse kinematics, no coupling.

Each of these is a documented next step rather than a hidden flaw. The repository keeps a running
[known-defects log](https://github.com/mulhamfetna/ros2-mediapipe-robotic-hand-digital-twin-vision-teleoperation/blob/main/docs/5-onshape-urdf/3-known-export-defects.md)
— including a mapping row that silently commanded 23° past a mechanical stop until it was found and
fixed, the structural change that makes that class of bug impossible, and two warnings ROS logs at
every startup that had gone unread for the life of the project.

## Cite it

```bibtex
@software{fetna_ros2_mediapipe_robotic_hand_2026,
  author    = {Fetna, Mulham Mohammed},
  title     = {{ROS 2 MediaPipe Robotic Hand: Real-Time Teleoperation Digital Twin}},
  year      = {2026},
  version   = {1.0.0},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.22658556},
  url       = {https://doi.org/10.5281/zenodo.22658556}
}
```
