---
title: "Finger joint angles from three hand landmarks: the dot-product geometry"
slug: "joint-angles-dot-product"
date: 2026-09-16
draft: false
description: "Compute finger joint angles from MediaPipe landmarks with a dot product: landmark triplets per knuckle, interior versus bend angle, a worked example, and the two numerical traps (zero-length vectors and arccos domain errors) with their Python guards."
keywords: ["joint angle from three points", "dot product angle between vectors", "MediaPipe finger angle", "hand landmark triplets", "arccos NaN numpy clip", "finger flexion angle python", "MCP PIP DIP angle"]
tags: ["computer-vision", "mathematics", "robotics", "python"]
categories: ["Projects"]
series: ["ROS 2 Tendon-Driven Hand MuJoCo Twin"]
series_order: 5
showDate: true
showAuthor: true
showTableOfContents: true
heroStyle: "background"
---

{{< katex >}}

{{< lead >}}
Every knuckle angle in this project comes from three landmarks and one dot product. No learning,
no lookup table — just the definition of the angle between two vectors, plus two guards that keep a
single glitchy frame from sending NaN into a motor command.
{{< /lead >}}

## Three points make an angle

An angle needs a vertex and two rays. A knuckle is the vertex; the two bones meeting there are the rays:

1. a **base point**, where the previous bone starts,
2. the **vertex** — the knuckle being measured,
3. an **end point**, where the next bone ends.

{{< mermaid >}}
flowchart LR
    P1(("p1<br>base")) -- "v1 = p1 − p2" --- P2(("p2<br>vertex<br>knuckle"))
    P2 -- "v2 = p3 − p2" --- P3(("p3<br>end"))
{{< /mermaid >}}

## The triplet table

```python
self.finger_triplets = {
    "thumb":  [(0, 1, 2), (1, 2, 3), (2, 3, 4)],
    "index":  [(0, 5, 6), (5, 6, 7), (6, 7, 8)],
    "middle": [(0, 9, 10), (9, 10, 11), (10, 11, 12)],
    "ring":   [(0, 13, 14), (13, 14, 15), (14, 15, 16)],
    "pinky":  [(0, 17, 18), (17, 18, 19), (18, 19, 20)]
}
```

![MediaPipe landmark indices used to build the triplets](mediapipe-landmarks.png "Landmark 0 — the wrist — starts every finger's first triplet.")

| Finger | Triplet | Vertex | Joint measured |
|---|---|---|---|
| Index | (0, 5, 6) | 5 | MCP — joins finger to palm |
| Index | (5, 6, 7) | 6 | PIP — middle knuckle |
| Index | (6, 7, 8) | 7 | DIP — fingertip knuckle |
| Thumb | (0, 1, 2) | 1 | CMC — saddle joint at the wrist |
| Thumb | (1, 2, 3) | 2 | MCP |
| Thumb | (2, 3, 4) | 3 | IP |

**Why every first triplet starts at 0.** The palm has no landmark of its own, so the wrist → knuckle
line stands in for the metacarpal bone. It isn't exactly collinear with a straight finger — ring and
pinky metacarpals fan outward — so a relaxed straight finger rarely measures a full \(\pi\). That's
part of why the calibrated "straight" threshold is 3.10 rad rather than 3.14.

## The function, line by line

```python
def _calculate_angle(self, p1, p2, p3):
    v1 = p1 - p2                    # ray back along the previous bone
    v2 = p3 - p2                    # ray out along the next bone
    norm1 = np.linalg.norm(v1)
    norm2 = np.linalg.norm(v2)
    if norm1 < 1e-6 or norm2 < 1e-6:
        return 0.0                  # trap 1: a zero-length bone
    cosang = np.dot(v1, v2) / (norm1 * norm2)
    cosang = np.clip(cosang, -1.0, 1.0)   # trap 2: floating-point overshoot
    return float(np.arccos(cosang))
```

It is the definition of the dot product, rearranged:

$$\cos\theta = \frac{\vec v_1 \cdot \vec v_2}{\lVert \vec v_1 \rVert\,\lVert \vec v_2 \rVert}
\qquad\Longrightarrow\qquad
\theta = \arccos\!\left(\frac{\vec v_1 \cdot \vec v_2}{\lVert \vec v_1 \rVert\,\lVert \vec v_2 \rVert}\right)$$

## Interior angle, not bend angle

The result is the **interior angle between the bones**, which runs the opposite way from the "bend" a
physiotherapist would quote:

| Finger | Bones | Interior \(\theta\) | Bend \(\pi - \theta\) |
|---|---|---|---|
| Perfectly straight | opposite directions | \(\pi\) = 180° | 0° |
| Right angle | perpendicular | \(\pi/2\) = 90° | 90° |
| Folded flat | same direction | 0 | 180° |

**Larger number, straighter finger.** Keep that inversion in mind — it's why the normalization in
[Part 7](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/flexion-normalization/)
subtracts from the *straight* limit.

## Worked example

An index finger, in normalized image coordinates (z omitted for readability):

| Landmark | x | y |
|---|---|---|
| 5 (MCP) | 0.50 | 0.60 |
| 6 (PIP) | 0.50 | 0.50 |
| 7 (DIP) | 0.55 | 0.45 |

For the PIP joint, triplet `(5, 6, 7)` with vertex 6:

$$\vec v_1 = p_5 - p_6 = (0.00,\ 0.10) \qquad \vec v_2 = p_7 - p_6 = (0.05,\ -0.05)$$

$$\vec v_1 \cdot \vec v_2 = -0.005 \qquad \lVert\vec v_1\rVert = 0.100 \qquad \lVert\vec v_2\rVert = 0.0707$$

$$\cos\theta = \frac{-0.005}{0.00707} = -0.707 \quad\Rightarrow\quad \theta = 2.356\ \text{rad} = 135^\circ$$

An interior angle of 135° — a 45° bend.

{{< alert icon="circle-info" >}}
These are image-normalized coordinates, so the true bend differs when the finger isn't aligned with
the image axes — up to 16° ([Part 4](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/mediapipe-hands/)).
MediaPipe's metric world landmarks remove that distortion.
{{< /alert >}}

## The two numerical traps

**1. Zero-length vectors.** If two landmarks coincide — a glitchy frame, or a finger foreshortened
straight at the lens — a norm is zero and the division yields `NaN`. The guard returns `0.0`. Note
what that means here: 0 rad is "folded flat", so a degenerate frame briefly reads as a curled joint.
After averaging and clipping, the cost is one frame of partial flexion instead of a crash.

**2. `arccos` outside its domain.** Rounding can produce a cosine of `1.0000000002`, and
`np.arccos` of that is `NaN`. `np.clip(cosang, -1, 1)` makes it impossible. Without the clip, a
perfectly straight finger intermittently produces `NaN`, which would propagate straight into the
tendon force.

Both guards are unglamorous and non-optional in anything that ends at an actuator.

## Next

One finger now has three angles — but it only has one tendon. [Part 6](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/underactuation-averaging/)
collapses them.
