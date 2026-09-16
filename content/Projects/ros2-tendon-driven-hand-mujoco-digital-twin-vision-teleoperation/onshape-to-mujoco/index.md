---
title: "From an Onshape assembly to a MuJoCo model with onshape-to-robot"
slug: "onshape-to-mujoco"
date: 2026-09-16
draft: false
description: "Export a tendon-driven robotic hand from Onshape CAD to MuJoCo MJCF with onshape-to-robot: the current SG90 servo design, how mates become joints, API keys, every config.json field (output_format, additional_xml, joint_properties, mergeSTLs), and CAD rules that make exports reliable."
keywords: ["onshape-to-robot", "Onshape to MuJoCo", "Onshape MJCF export", "CAD to simulation robotics", "onshape-to-robot config.json", "additional_xml tendons", "robotic hand CAD design", "SG90 servo hand"]
tags: ["robotics", "cad", "mujoco", "onshape", "mechatronics"]
categories: ["Projects"]
series: ["ROS 2 Tendon-Driven Hand MuJoCo Twin"]
series_order: 12
showDate: true
showAuthor: true
showTableOfContents: true
heroStyle: "background"
---

{{< lead >}}
The simulated hand was never modelled by hand. It is an Onshape assembly — five SG90 servos, fifteen
knuckle mates, a palm full of tendon channels — pulled through the Onshape API and written out as
MuJoCo XML. Here is the design, and every setting that steers the export.
{{< /lead >}}

{{< button href="https://cad.onshape.com/documents/a2dbb5f16624f10f1aa22f02/w/693ffc0be3e83ae21f78f6ed/e/4d69727744037003575f4068?renderMode=0" target="_blank" >}}Open the Onshape assembly{{< /button >}}

## The design

{{< video src="onshape_cad_tour_current_design.webm" controls="true" muted="true" playsinline="true" >}}

{{< gallery >}}
  <img src="current_assembly_iso.png" class="grid-w50" alt="Current assembly, isometric, with mate connectors visible" />
  <img src="current_front_tendon_channels.png" class="grid-w50" alt="Front view: one tendon channel per finger down the palm" />
  <img src="current_servo_block.png" class="grid-w50" alt="Five SG90 servos staggered in the base block" />
  <img src="current_servo_layout.png" class="grid-w50" alt="Servo layout seen from below the palm" />
  <img src="current_fingers_flexing.png" class="grid-w50" alt="Finger and thumb mates driven to flexion in Onshape" />
  <img src="current_side_profile_flexed.png" class="grid-w50" alt="Side profile of the flexed finger and thumb chains" />
{{< /gallery >}}

- **Palm and fingers:** four three-phalanx fingers and a three-segment thumb, every knuckle a
  **revolute mate** with limits. The RGB triads in the views are mate connectors.
- **Tendon channels:** one per finger, running down the palm into the base.
- **Servo block:** five SG90-class servos, staggered so each horn sits under a tendon exit.

## The design has a history

{{< timeline >}}
{{< timelineItem icon="pencil" header="Start" subheader="2026-09-02" md="true" >}}
The first version in the history.
{{< /timelineItem >}}
{{< timelineItem icon="star" header="v1.0.0 — MediaPipe" subheader="2026-09-06" md="true" >}}
The joint-angle-driven hand behind the [RViz predecessor project](/projects/ros2-mediapipe-robotic-hand-digital-twin-vision-teleoperation/).
{{< /timelineItem >}}
{{< timelineItem icon="edit" header="v1.0.1 → Main" subheader="2026-09-08" md="true" >}}
Point release and the main line the later work branches from.
{{< /timelineItem >}}
{{< timelineItem icon="fork" header="V3 → Mujoco branch" subheader="2026-09-16" md="true" >}}
The current design used by this twin — the version with the servo base block shown above.
{{< /timelineItem >}}
{{< /timeline >}}

| Onshape version history | Mate features |
|---|---|
| ![Onshape version graph: Start, v1.0.0 MediaPipe, v1.0.1, Main, V3, Mujoco branch](current_version_history.png) | ![Mate features list: dof_pinky_dip through dof_thumb_ip, then Fastened mates](current_mate_features.png) |

*43 part instances, 112 mate features: the 15 `dof_*` knuckle mates, the servo mates, and many `Fastened` mates.*

![An early assembly from 2026-09-09, before the servo base block](onshape_assembly_back.png "An earlier revision (2026-09-09), before the servo base existed.")

## Mates become joints

onshape-to-robot turns each **mate** into a MuJoCo **joint** and keeps its **name**, stripping the
`dof_` prefix — `dof_index_pip` becomes joint `index_pip`. Naming in CAD is part of the software
interface.

![The 15 knuckle mates in Onshape's feature tree](onshape_mate_features.png "Early revision's mate list — the same dof_* names survive into robot.xml.")

The model ends up with 20 hinge joints: 15 knuckles and 5 servo horns, with limits taken from the mates.

## Running the export

```bash
pip install onshape-to-robot                        # any recent Python; separate env is fine

# Onshape API keys — create at https://dev-portal.onshape.com/keys, keep them in a git-ignored .env
export ONSHAPE_API=https://cad.onshape.com
export ONSHAPE_ACCESS_KEY=...
export ONSHAPE_SECRET_KEY=...

onshape-to-robot mujoco_twin/model                  # the directory holding config.json
```

It writes `robot.xml` plus `assets/*.stl` (and `.part` metadata) into that directory.

{{< alert icon="triangle-exclamation" >}}
**The export overwrites `robot.xml`.** The joint defaults and the `<actuator>` block are manual edits
that must be re-applied ([Part 13](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/mjcf-edits/)).
Commit first, so `git diff` shows exactly what changed.
{{< /alert >}}

## `config.json`, field by field

```json
{
  "url": "https://cad.onshape.com/documents/a2dbb5f16624f10f1aa22f02/w/693ffc0be3e83ae21f78f6ed/e/4d69727744037003575f4068",
  "output_format": "mujoco",
  "clearance": 0.0,
  "ignoreLimits": false,
  "useMeshes": true,
  "mergeSTLs": "no",
  "additional_xml": "tendons.xml",
  "joint_properties": {
    "default": { "actuated": false },
    "servo_index":  { "actuated": true },
    "servo_middle": { "actuated": true },
    "servo_ring":   { "actuated": true },
    "servo_pinky":  { "actuated": true },
    "servo_thumb":  { "actuated": true }
  }
}
```

| Field | Value | What it does | Why it matters for a tendon hand |
|---|---|---|---|
| `url` | assembly URL | document / workspace / element to export | a `…/w/…` workspace URL exports the *current* state; use `…/v/…` for a frozen, reproducible export |
| `output_format` | `"mujoco"` | MJCF instead of the default URDF | MJCF can express tendons, sites and tendon actuators; URDF can't |
| `additional_xml` | `"tendons.xml"` | injects a file into the output | tendons and contacts survive every re-export (`robot.xml` carries `<!-- Additional tendons.xml -->`) |
| `clearance` | `0.0` | padding on collision geometry | 1:1 CAD tolerances — intersecting parts show up instead of hiding |
| `ignoreLimits` | `false` | keep mate limits as joint ranges | the 90° knuckle stops come from here, and the tendon physics depends on them |
| `useMeshes` | `true` | STL meshes for geoms | phalanges keep their real shape |
| `mergeSTLs` | `"no"` | one STL per part | per-part mass and inertia stay accurate for 1.8–5.4 g phalanges |

### `joint_properties`: keep the knuckles passive

By default the exporter actuates **every** joint — 20 motors, one inside each knuckle.
`"default": {"actuated": false}` makes the knuckles **passive**, so only tendons can move them —
which is what an underactuated hand is. The `servo_*` entries mark the horn joints as actuated, but in
the committed model those generated servo actuators are replaced by hand with five linear tendon
motors, and the horns spin freely.

## CAD rules that make exports work

- **Name mates as you want joints named.** Code and XML address joints and sites by name.
- **Set limits on every knuckle — and mind the sign.** The mate's axis direction is exported as-is,
  so some joints come out `[−90°, 0]` and others `[0, +90°]`. This model has both
  ([Part 14](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/model-anatomy/)).
- **Define the tendon via-points in CAD** as reference frames the exporter turns into `<site>`s,
  following its frame-naming convention ([documentation](https://onshape-to-robot.readthedocs.io/)).
  A renamed frame fails at load time with `Error: site '…' not found in wrap N`.
- **Avoid interpenetrating parts** at the export pose.
