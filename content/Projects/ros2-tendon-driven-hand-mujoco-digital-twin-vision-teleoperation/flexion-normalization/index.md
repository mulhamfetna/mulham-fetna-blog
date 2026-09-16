---
title: "Normalizing finger curl: from radians to a 0–1 flexion"
slug: "flexion-normalization"
date: 2026-09-16
draft: false
description: "Inverted min-max normalization turns an average knuckle angle into a 0–1 finger flexion for robot control: the formula, why it subtracts from the straight limit, why clipping matters, where the 3.10 and 1.60 rad constants come from, and why flexion is what travels over ROS 2."
keywords: ["min max normalization", "inverted normalization formula", "finger flexion normalization", "hand tracking calibration constants", "normalize joint angle 0 to 1", "robot teleoperation mapping", "np.clip normalization"]
tags: ["computer-vision", "mathematics", "robotics", "python"]
categories: ["Projects"]
series: ["ROS 2 Tendon-Driven Hand MuJoCo Twin"]
series_order: 7
showDate: true
showAuthor: true
showTableOfContents: true
heroStyle: "background"
---

{{< katex >}}

{{< lead >}}
Radians belong to the camera. Newtons belong to the robot. The number that crosses between them is a
plain percentage — and producing it takes one inverted formula and one clip.
{{< /lead >}}

## The formula

```python
RAW_STRAIGHT_ANGLE = 3.10  # ~177.6° — open finger
RAW_CURLED_ANGLE   = 1.60  # ~91.7° — fully curled finger

flexion = (straight_limit - avg_angle) / (straight_limit - curled_limit)
flexions[finger_name] = float(np.clip(flexion, 0.0, 1.0))
```

$$\text{flexion} = \operatorname{clip}\!\left(\frac{\theta_\text{straight} - \bar\theta}{\theta_\text{straight} - \theta_\text{curled}},\; 0,\; 1\right)$$

{{< chart >}}
type: 'line',
data: {
  labels: ['3.30','3.20','3.10','3.00','2.90','2.80','2.70','2.60','2.50','2.40','2.30','2.20','2.10','2.00','1.90','1.80','1.70','1.60','1.50','1.40','1.30'],
  datasets: [
    { label: 'Index–pinky window (3.10 → 1.60 rad)', data: [0,0,0,0.067,0.133,0.2,0.267,0.333,0.4,0.467,0.533,0.6,0.667,0.733,0.8,0.867,0.933,1,1,1,1], borderColor: '#2a78d6', backgroundColor: '#2a78d6', borderWidth: 2, pointRadius: 2, tension: 0 },
    { label: 'Thumb window (2.90 → 2.30 rad)', data: [0,0,0,0,0,0.167,0.333,0.5,0.667,0.833,1,1,1,1,1,1,1,1,1,1,1], borderColor: '#eb6834', backgroundColor: '#eb6834', borderWidth: 2, pointRadius: 2, tension: 0 }
  ]
},
options: {
  plugins: { title: { display: true, text: 'Average knuckle angle → normalized flexion' } },
  scales: {
    x: { title: { display: true, text: 'average knuckle angle (rad) — straight → curled' } },
    y: { min: 0, max: 1.05, title: { display: true, text: 'flexion (0 open · 1 closed)' } }
  }
}
{{< /chart >}}

## The three pieces

**Denominator — the range.** \(3.10 - 1.60 = 1.50\) rad of travel. It sets the scale.

**Numerator — the distance travelled** away from straight. At \(\bar\theta = 2.35\): \(3.10 - 2.35 = 0.75\) rad.

**Division — the percentage.** \(0.75 / 1.50 = 0.5\): half the range used.

## Why "backwards"

Textbook min-max normalization is \((x - \min)/(\max - \min)\). This one subtracts from the
**maximum**, because the interior angle
([Part 5](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/joint-angles-dot-product/))
runs opposite to the output we want:

| Finger | Interior angle | Wanted flexion |
|---|---|---|
| Open | large (3.10) | 0.0 |
| Closed | small (1.60) | 1.0 |

Check the extremes:

$$\frac{3.10 - 3.10}{1.50} = 0.0 \qquad \frac{3.10 - 2.35}{1.50} = 0.5 \qquad \frac{3.10 - 1.60}{1.50} = 1.0$$

## Why clip

Real hands overshoot the window: hyperextended fingers exceed 3.10 rad, a hard fist dips under 1.60.
Unclipped, flexion leaves \([0, 1]\) and the force mapping would command beyond the actuator's range.
MuJoCo would clamp it anyway (`ctrlrange`), but clipping at the source keeps the **topic** honest for
every other subscriber.

A side effect you get for free: the flat regions of the chart are small dead zones at both ends,
which hide jitter when the hand is already fully open or closed.

## Where 3.10 and 1.60 come from

**They are measured, not derived** — read off a live system with one person's open hand and fist in
front of one webcam. They silently absorb:

- that person's hand anatomy,
- MediaPipe's model bias (a straight finger rarely reads exactly \(\pi\)),
- the aspect-ratio skew of normalized coordinates ([Part 4](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/mediapipe-hands/)),
- the orientation the hand is usually held at.

Change any of those and the window drifts. A recorded live session shows exactly how far: that
operator's fist only reached flexion **0.67–0.76** —
[Part 9](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/calibration/).

## Normalize here, interpolate there

Normalization and linear interpolation are inverses, and the pipeline puts one on each side of the
network:

| Step | Operation | In → out | Runs in |
|---|---|---|---|
| Vision | **normalize** | radians → 0..1 | `vision_tracker` |
| Actuation | **lerp** | 0..1 → newtons | `mujoco_twin` |

Sending **0..1** over ROS 2 is what keeps the sides independent: recalibrate vision without touching
the simulator, retune the simulator (or swap in real servos) without touching vision.
[Part 10](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/flexion-to-force/)
is the other half.
