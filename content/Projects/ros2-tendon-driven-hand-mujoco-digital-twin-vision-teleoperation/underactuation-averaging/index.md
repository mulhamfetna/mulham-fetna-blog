---
title: "Underactuation: why three knuckle angles become one tendon command"
slug: "underactuation-averaging"
date: 2026-09-16
draft: false
description: "Mapping a human finger's three joints onto a one-tendon underactuated robotic finger: why the knuckle angles are averaged across space (not time), how claw and right-angle poses compare, what averaging discards, and what changes with more tendons."
keywords: ["underactuated robotic hand", "underactuation", "tendon driven finger control", "human to robot hand mapping", "hand retargeting", "averaging joint angles", "one tendon three joints"]
tags: ["robotics", "mechatronics", "computer-vision", "mujoco"]
categories: ["Projects"]
series: ["ROS 2 Tendon-Driven Hand MuJoCo Twin"]
series_order: 6
showDate: true
showAuthor: true
showTableOfContents: true
heroStyle: "background"
---

{{< katex >}}

{{< lead >}}
`avg_angle = np.mean(angles)` looks like noise filtering. It isn't. It is a mechanical decision: a
human finger has three joints you can move separately, and this robot finger has one string.
{{< /lead >}}

## Not smoothing — compression

"Averaging" in a sensor pipeline usually means averaging over **time** to reduce noise. Nothing here
keeps history between frames. The mean is taken **across space** — over the three joints of one
finger in a single frame — to solve a problem called **underactuation**.

## The 3-to-1 problem

| | Human finger | Robot finger |
|---|---|---|
| Joints | 3 (MCP, PIP, DIP) | 3 hinges (`*_mcp`, `*_pip`, `*_dip`) |
| Independent actuators | many muscles; joints move semi-independently | **1 flexor tendon, 1 motor** |
| Degrees of freedom you can command | ~3 | **1** |

A system with fewer actuators than joints is **underactuated**. A single tendon threads all three
joints of each robot finger, so the only command is *"pull this string with force F"* — and the
joints share that pull according to routing geometry and dynamics.

![The index finger's single flexor tendon threaded through six labelled sites from palm to fingertip](tendon_routing_index.png "One string, six via-points, three joints. Pull it and all three knuckles move together.")

So the vision layer must compress **three human measurements into one robot command**.

## Why the mean and not one joint

Use only the base (MCP) knuckle and two poses break apart:

- **Right angle** — bent at the base, fingers straight: MCP ≈ 90°, the twin closes. ✔
- **Claw** — base straight, tips curled: MCP ≈ 180°, the twin stays **flat** while the finger is
  visibly curled. ✘

With the mean of all three, using the finger window 3.10 → 1.60 rad from
[Part 7](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/flexion-normalization/):

| Human pose | MCP | PIP | DIP | Mean \(\bar\theta\) | Flexion |
|---|---|---|---|---|---|
| Open hand | 3.10 | 3.10 | 3.10 | **3.10** | 0.00 |
| Claw (tips only) | 3.10 | 2.20 | 2.20 | **2.50** | 0.40 |
| Right angle (base only) | 1.60 | 3.10 | 3.10 | **2.60** | 0.33 |
| Full fist | 1.60 | 1.60 | 1.60 | **1.60** | 1.00 |

{{< chart >}}
type: 'bar',
data: {
  labels: ['Open hand', 'Claw (tips only)', 'Right angle (base only)', 'Full fist'],
  datasets: [
    { label: 'MCP only as the signal', data: [0.00, 0.00, 1.00, 1.00], backgroundColor: 'rgba(235, 104, 52, 0.75)' },
    { label: 'Mean of MCP, PIP, DIP', data: [0.00, 0.40, 0.33, 1.00], backgroundColor: 'rgba(42, 120, 214, 0.75)' }
  ]
},
options: {
  plugins: { title: { display: true, text: 'Flexion produced by four poses: one joint vs the mean' } },
  scales: { y: { min: 0, max: 1, title: { display: true, text: 'flexion (0 open · 1 closed)' } } }
}
{{< /chart >}}

Any curl anywhere on the finger lowers the mean, so the twin responds to every way a person closes a
finger — and a tight fist gives the strongest pull.

## What the mean throws away

Averaging is lossy on purpose:

- **Pose shape.** "Claw" and "right angle" land close together. With one tendon the robot couldn't
  reproduce the difference anyway, so nothing that matters is lost.
- **Weighting is a choice.** In a human grasp the MCP does most of the closing; a weighted mean such
  as \(0.5\,\theta_\text{MCP} + 0.3\,\theta_\text{PIP} + 0.2\,\theta_\text{DIP}\) is a valid
  refinement, but the weights would need tuning against the robot's real joint coupling.
- **How the robot splits the pull isn't decided here.** That depends on moment arms and joint
  dynamics. In this model it comes out almost identical across the three index joints —
  [Part 11](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/tendon-physics-switch/).

![Only the index tendon pulled: all three index joints curl together](isolate_index.png "One tendon, one command: the whole index finger curls as a unit.")

## With more tendons

A future hardware revision with two tendons per finger — one for MCP, one shared by PIP and DIP —
makes the compression 3 → 2: average PIP and DIP (which are anatomically coupled in humans too) and
send MCP on its own. The flexion topic just grows from five numbers to ten.
