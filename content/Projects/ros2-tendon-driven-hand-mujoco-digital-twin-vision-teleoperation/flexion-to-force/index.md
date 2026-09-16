---
title: "From finger flexion to tendon force: linear interpolation onto a MuJoCo motor"
slug: "flexion-to-force"
date: 2026-09-16
draft: false
description: "How a 0–1 finger flexion becomes a ±50 N MuJoCo tendon motor command: the lerp formula, actuator lookup by name with mj_name2id (and its silent -1 failure), what a positive 'pushing' tendon force means, and why ctrlrange must match the Python constants."
keywords: ["linear interpolation lerp", "MuJoCo motor actuator tendon", "mj_name2id", "MuJoCo data.ctrl", "MuJoCo ctrlrange", "tendon force control", "flexion to force mapping"]
tags: ["robotics", "mujoco", "mathematics", "python"]
categories: ["Projects"]
series: ["ROS 2 Tendon-Driven Hand MuJoCo Twin"]
series_order: 10
showDate: true
showAuthor: true
showTableOfContents: true
heroStyle: "background"
---

{{< katex >}}

{{< lead >}}
On the far side of the ROS 2 topic, five flexions arrive and five tendon motors wait. One line of
linear interpolation connects them — plus a name lookup that can fail silently, and a string that is
allowed to push.
{{< /lead >}}

## The formula

$$F(t) = F_\text{open} + t\,(F_\text{closed} - F_\text{open}) = 50 + t\,(-50 - 50) = 50 - 100\,t$$

```python
FORCE_OPEN = 50.0
FORCE_CLOSED = -50.0

def _lerp(self, start_val, end_val, t):
    return start_val + t * (end_val - start_val)

def apply_flexions(self, flexions):
    for finger, flexion_amount in flexions.items():
        target_force = self._lerp(FORCE_OPEN, FORCE_CLOSED, flexion_amount)
        self.data.ctrl[self.motors[finger]] = target_force
```

{{< chart >}}
type: 'line',
data: {
  labels: ['0.0','0.1','0.2','0.3','0.4','0.5','0.6','0.7','0.8','0.9','1.0'],
  datasets: [{
    label: 'Tendon motor command (N)',
    data: [50, 40, 30, 20, 10, 0, -10, -20, -30, -40, -50],
    borderColor: '#2a78d6', backgroundColor: '#2a78d6', borderWidth: 2, pointRadius: 3, tension: 0
  }]
},
options: {
  plugins: { legend: { display: false }, title: { display: true, text: 'Flexion → force: +50 N open, 0 N at 0.5, −50 N closed' } },
  scales: { x: { title: { display: true, text: 'flexion' } }, y: { title: { display: true, text: 'force (N) — negative pulls' } } }
}
{{< /chart >}}

## Three parts of one line

With \(t = 0.75\), a finger 75% closed:

1. **Range:** \(F_\text{closed} - F_\text{open} = -100\) N — the whole span of the scale.
2. **Distance along it:** \(0.75 \times -100 = -75\) N.
3. **Anchor:** the scale starts at +50, not 0, so \(50 + (-75) = -25\) N.

| Flexion \(t\) | Force | Meaning |
|---|---|---|
| 0.00 | **+50 N** | open — the motor pushes the tendon |
| 0.50 | **0 N** | no actuation |
| 1.00 | **−50 N** | fist — maximum pull |

## Seen live

![OK sign, live: pull_thumb −32.9 N and pull_index −9.67 N; the other fingers near +46 N](live_ok_sign.jpg "MuJoCo's Control panel during an OK sign: thumb flexion 0.83 → −32.9 N, index 0.60 → −9.7 N, open fingers near +46 N.")

## Looking actuators up by name

```python
self.motors = {
    "thumb":  mujoco.mj_name2id(self.model, mujoco.mjtObj.mjOBJ_ACTUATOR, "pull_thumb"),
    "index":  mujoco.mj_name2id(self.model, mujoco.mjtObj.mjOBJ_ACTUATOR, "pull_index"),
    ...
}
```

`data.ctrl` is a flat array ordered by the **XML** — here pinky, ring, middle, index, thumb (IDs 0–4).
Resolving names once at startup means reordering or adding actuators never breaks the Python.

{{< alert icon="triangle-exclamation" >}}
**The silent failure.** Rename an actuator and `mj_name2id` returns `-1` — no exception. Then
`data.ctrl[-1]` writes to the **last** actuator: one finger's command drives another finger. Assert
that every ID is `>= 0` at startup.
{{< /alert >}}

On the ROS side, incoming names are filtered against the same dictionary, so an unknown finger name
in a message is ignored instead of crashing the callback:

```python
flexions = {name: pos for name, pos in zip(msg.name, msg.position) if name in self.sim.motors}
```

## A string that pushes

A MuJoCo `<motor>` on a tendon applies force along the tendon's length (gear = 1):

- **negative** → shortens the tendon → **pulls**, exactly what a servo winding a string does;
- **positive** → lengthens it → **pushes**.

Real strings can't push. MuJoCo lets a tendon motor apply compression anyway, and this model uses
that to drive fingers open at +50 N — which presses them about **3° past their open stop** into the
soft joint limit. On hardware the equivalent is a second, antagonistic tendon or a return spring. The
model *has* extensor tendons, but they're passive — and that has consequences
([Part 11](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/tendon-physics-switch/)).

## `ctrlrange` must agree with Python

```xml
<motor name="pull_index" tendon="tendon_flex_index" ctrlrange="-50 50" .../>
```

The limits live in two files. MuJoCo clamps `ctrl` to `ctrlrange` (the model uses
`autolimits="true"`), so raising `FORCE_CLOSED` to −80 without editing the XML silently saturates at −50.
