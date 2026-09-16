---
title: "Inside the MuJoCo tendon hand model: bodies, joints, sites and sign conventions"
slug: "model-anatomy"
date: 2026-09-16
draft: false
description: "A reference tour of a MuJoCo tendon-driven hand model: 22 bodies, 20 hinge joints, 70 sites, 10 tendons and 5 actuators; the kinematic tree, inconsistent joint sign conventions from CAD, the site naming scheme, tendon rest lengths, and visual versus collision geoms."
keywords: ["MuJoCo model structure", "MuJoCo kinematic tree", "MuJoCo joint range sign", "MuJoCo sites naming", "MuJoCo tendon length", "MuJoCo visual collision geom groups", "robotic hand model reference"]
tags: ["robotics", "mujoco", "simulation", "reference"]
categories: ["Projects"]
series: ["ROS 2 Tendon-Driven Hand MuJoCo Twin"]
series_order: 14
showDate: true
showAuthor: true
showTableOfContents: true
heroStyle: "background"
---

{{< lead >}}
A fixed palm, five three-segment digits, five free-spinning servo horns, ten strings through seventy
points, five motors. Every number here was read from the compiled model with MuJoCo 3.13.0.
{{< /lead >}}

{{< carousel images="{pose_open_front.png,pose_neutral_front.png,pose_fist_front.png,isolate_index.png,isolate_middle.png,isolate_ring.png,isolate_pinky.png,isolate_thumb.png}" aspectRatio="4-3" interval="2500" captions="{pose_open_front.png:Open (+50 N),pose_neutral_front.png:Neutral (0 N),pose_fist_front.png:Fist (−50 N),isolate_index.png:Index tendon only,isolate_middle.png:Middle tendon only,isolate_ring.png:Ring tendon only,isolate_pinky.png:Pinky tendon only,isolate_thumb.png:Thumb tendon only}" >}}

## At a glance

| Quantity | Value | Notes |
|---|---|---|
| Bodies | 22 | world, palm, 15 phalanges, 5 servo horns |
| Joints / DoF | 20 / 20 | all hinges; **no free joint** — the palm is welded to the world |
| Knuckle joints | 15 | passive, 90° limits |
| Servo horn joints | 5 | unlimited and unactuated |
| Geoms | 87 | 43 visual + 43 collision meshes + floor |
| Meshes | 14 | STL |
| Sites | 70 | tendon via-points and anchors |
| Tendons | 10 | 5 flexor (actuated) + 5 extensor (passive) |
| Actuators | 5 | `<motor>` on flexors, ±50 N |
| Timestep / integrator | 0.002 s / Euler | 500 steps per simulated second |
| Total mass | 0.398 kg | palm block 341.8 g |

## The kinematic tree

{{< mermaid >}}
flowchart TB
    W["world"] --> P["part_1 · palm + servo block<br>341.8 g · 20 sites"]
    P --> I1["part_2_4 · index proximal<br>4.6 g · index_mcp"] --> I2["part_3_4<br>1.8 g · index_pip"] --> I3["part_4_4<br>4.6 g · index_dip"]
    P --> M1["part_2_3 · middle<br>middle_mcp"] --> M2["part_3_3<br>middle_pip"] --> M3["part_4_3<br>middle_dip"]
    P --> R1["part_2_2 · ring<br>ring_mcp"] --> R2["part_3_2<br>ring_pip"] --> R3["part_4_2<br>ring_dip"]
    P --> K1["part_2 · pinky<br>pinky_mcp"] --> K2["part_3<br>pinky_pip"] --> K3["part_4<br>pinky_dip"]
    P --> T1["part_5 · thumb<br>5.4 g · thumb_cmc"] --> T2["part_6<br>2.4 g · thumb_mp"] --> T3["part_7<br>2.5 g · thumb_ip"]
    P --> H["servo_horn … servo_horn_5<br>0.3 g each"]
{{< /mermaid >}}

{{< alert icon="triangle-exclamation" >}}
**Body names are CAD part names, not finger names.** Repeated instances get `_2`, `_3`, … appended, so
the *pinky* chain is `part_2 → part_3 → part_4` and the *index* chain is `part_2_4 → part_3_4 → part_4_4`.
Address **joints, sites, tendons and actuators** in code — never bodies.
{{< /alert >}}

## Joints and their signs

| Joint | Range (rad) | Bend direction |
|---|---|---|
| `index_mcp`, `index_pip`, `index_dip` | [−1.571, 0] | negative |
| `middle_mcp` | [−1.571, 0] | negative |
| `middle_pip`, `middle_dip` | [0, +1.571] | **positive** |
| `ring_mcp`, `ring_dip` | [−1.571, 0] | negative |
| `ring_pip` | [0, +1.571] | **positive** |
| `pinky_mcp`, `pinky_dip` | [−1.571, 0] | negative |
| `pinky_pip` | [0, +1.571] | **positive** |
| `thumb_cmc`, `thumb_mp`, `thumb_ip` | [−1.571, 0] | negative |
| `servo_*` × 5 | unlimited | — |

**Bend direction is inconsistent**, because each Onshape mate's axis was exported as-is. Anything that
reads joint angles must normalize per joint:

```python
direction = 1.0 if model.jnt_range[jid][1] > 1e-6 else -1.0   # [0, +90°] → +1 ; [-90°, 0] → -1
bend_deg = np.degrees(direction * data.qpos[model.jnt_qposadr[jid]])
```

Readings just beyond the range — 94° at −50 N, −3° at +50 N — are MuJoCo **soft limits**: constraint
force grows with penetration rather than acting as a rigid wall.

## Site naming

Every site follows `{segment}_{role}_{finger}`:

| Segment | Role | Finger |
|---|---|---|
| `horn` · `palm_in` · `palm_out` · `proximal` · `intermediate_in` · `intermediate_out` · `anchor` | `flex` · `ext` | `thumb` · `index` · `middle` · `ring` · `pinky` |

7 × 2 × 5 = **70 sites**: 20 on the palm, 2 per proximal phalanx, 4 per intermediate, 2 per distal,
2 per servo horn.

| The index flexor's route | Every site on the hand |
|---|---|
| ![Index flexor via-points labelled](tendon_routing_index.png) | ![All tendon sites rendered](tendon_sites_overview.png) |

## Tendons at rest

| Tendon (flexor / extensor) | Rest length (m) | Stiffness (N/m) | Damping |
|---|---|---|---|
| index | 0.1283 / 0.1283 | 15 | 0.05 |
| middle | 0.1260 / 0.1260 | 15 | 0.05 |
| ring | 0.1245 / 0.1245 | 15 | 0.05 |
| pinky | 0.1241 / 0.1241 | 15 | 0.05 |
| thumb | 0.0843 / 0.0844 | 15 | 0.05 |

The index flexor measures **0.1294 m** pushed open and **0.0870 m** in a full fist — a **42 mm
stroke**, which is what a hardware servo horn would have to wind.

## Visual and collision geoms

| Class | `group` | `contype` / `conaffinity` | Visible by default |
|---|---|---|---|
| `visual` | 2 | 0 / 0 | yes |
| `collision` | 3 | 1 / 0 | no — toggle group 3 |

Both use the same STL meshes. Because robot collision geoms can never match each other, there is **no
self-contact** ([Part 13](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/mjcf-edits/)).

## Poke it yourself

```bash
env -u PYTHONPATH venv/bin/python -m mujoco.viewer --mjcf=mujoco_twin/model/scene.xml
```

**Rendering → Tendon** shows the strings, **site groups** show the via-points, **geom group 3** shows
collision meshes, and the **Control** panel drives `pull_*` by hand — the same panel the ROS node drives
live:

![MuJoCo Control panel driven live by the ROS node during an OK sign](live_ok_sign.jpg "Right: the Control panel, here written by the ROS 2 node rather than by a mouse.")
