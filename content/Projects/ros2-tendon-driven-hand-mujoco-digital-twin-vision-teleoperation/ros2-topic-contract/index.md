---
title: "The ROS 2 topic contract: one JointState topic, two nodes, and rclpy inside a render loop"
slug: "ros2-topic-contract"
date: 2026-09-16
draft: false
description: "Designing the ROS 2 interface of a vision-to-simulation pipeline: why a stock sensor_msgs/JointState carries normalized flexions on /hand/target_flexions, a timer-driven publisher, a subscriber pumped with rclpy.spin_once inside the MuJoCo passive viewer loop, and CLI recipes to observe and drive it."
keywords: ["ROS 2 JointState topic", "rclpy spin_once render loop", "MuJoCo ROS 2 integration", "rclpy timer publisher", "ros2 topic pub JointState", "ROS 2 Jazzy Python node", "mujoco viewer launch_passive rclpy"]
tags: ["ros2", "robotics", "mujoco", "python"]
categories: ["Projects"]
series: ["ROS 2 Tendon-Driven Hand MuJoCo Twin"]
series_order: 15
showDate: true
showAuthor: true
showTableOfContents: true
heroStyle: "background"
---

{{< lead >}}
The whole ROS 2 layer is one topic carrying five numbers. The interesting decisions are what those
numbers mean, which message type carries them, and how to run a ROS subscriber when a 3D viewer owns
your main thread.
{{< /lead >}}

## Why ROS 2 between vision and physics at all?

The single-process script works well. Splitting it across ROS 2 buys:

| Benefit | Concretely |
|---|---|
| **Process isolation** | a MediaPipe crash doesn't kill the simulator, and vice versa |
| **Independent environments** | vision and physics get their own container, dependencies and restarts |
| **Swappable endpoints** | replace the twin with a servo driver, or the tracker with a data glove |
| **Free observability** | `ros2 topic echo`, `hz`, `bag record` on the live stream |
| **Network transparency** | any machine on the LAN can subscribe |

The costs — a bigger stack, DDS configuration, one more hop — are small next to 19–85 ms of inference
([Part 19](/projects/ros2-tendon-driven-hand-mujoco-digital-twin-vision-teleoperation/latency-benchmark/)).

## The node graph

{{< mermaid >}}
flowchart LR
    subgraph C1["🐳 vision_tracker"]
        V["/vision_tracker_node<br>timer · 30 Hz"]
    end
    subgraph C2["🐳 mujoco_twin"]
        T["/mujoco_twin_node<br>spin_once in viewer loop"]
    end
    V -- "/hand/target_flexions<br>sensor_msgs/JointState · depth 10" --> T
    V -. "any LAN subscriber<br>ROS_DOMAIN_ID=42" .-> X["ros2 topic echo · rosbag<br>future servo driver"]
{{< /mermaid >}}

![Peace sign live: the tracked hand, the twin, and the forces produced from the topic's flexions](live_peace_sign.jpg "Every frame of the twin is driven by one JointState message like the one below.")

## The contract

| Field | Value |
|---|---|
| Topic | `/hand/target_flexions` |
| Type | `sensor_msgs/msg/JointState` |
| Publisher | `vision_tracker_node`, timer at 30 Hz — effective rate bounded by inference |
| Subscriber | `mujoco_twin_node` |
| QoS | default reliable, keep-last **10** |
| `header.stamp` | publisher clock at publish time |
| `name` | `["thumb", "index", "middle", "ring", "pinky"]` |
| `position` | flexion per finger, **0.0 open … 1.0 closed**, same order as `name` |
| `velocity`, `effort` | empty |

A real message, captured with `ros2 topic echo` during testing (published by hand with `ros2 topic pub`,
hence the zero stamp):

```yaml
header:
  stamp: {sec: 0, nanosec: 0}
  frame_id: ''
name: [thumb, index, middle, ring, pinky]
position: [0.1, 0.2, 0.3, 0.4, 0.5]
velocity: []
effort: []
```

### Why `JointState` — and why flexions, not radians

- **A stock message means no custom package** — no `colcon build`, no `rosidl` step, no message
  definitions duplicated across containers, and every ROS tool already understands it.
- **`position` holds unitless flexions, not joint radians.** The names are fingers, not model joints.
  That keeps the vision side ignorant of the robot's joint structure.
- **Don't remap it onto `/joint_states`** — `robot_state_publisher` would treat the values as radians
  for joints that don't exist.
- **Names travel with values.** The subscriber zips `name` and `position`, so order doesn't matter and
  unknown names are ignored.

## The publisher: timer-driven

```python
class VisionTrackerNode(Node):
    def __init__(self):
        super().__init__("vision_tracker_node")
        self.tracker = HandTracker(CAMERA_INDEX)
        self.publisher = self.create_publisher(JointState, TOPIC, 10)
        self.timer = self.create_timer(PUBLISH_PERIOD, self.timer_callback)   # 1/30 s

    def timer_callback(self):
        ret, frame = self.tracker.cap.read()
        ...
        flexions, annotated_frame = self.tracker.get_finger_flexions(frame)
        msg = JointState()
        msg.header.stamp = self.get_clock().now().to_msg()
        msg.name = FINGERS
        msg.position = [flexions[f] for f in FINGERS]
        self.publisher.publish(msg)
        cv2.imshow("MediaPipe Hand Tracker", annotated_frame)
        if cv2.waitKey(1) & 0xFF == 27:
            raise KeyboardInterrupt     # ESC shuts the node down cleanly
```

- **Single-threaded executor.** When a callback overruns its 33 ms period — as it does at ~85 ms with
  the viewer open — ticks don't pile up into a backlog of stale frames.
- **The camera read lives in the callback.** Simple and correct at this rate; production would grab
  frames on a thread and always process the newest.
- **Knobs:** `CAMERA_INDEX` (default 0) and `VISION_RATE_HZ` (default 30), set from Compose.

## The subscriber: pumped from inside a render loop

```python
def main():
    rclpy.init()
    sim = DigitalTwin(SCENE_XML)
    node = MujocoTwinNode(sim)
    with mujoco.viewer.launch_passive(sim.model, sim.data) as viewer:
        start_time = time.time()
        while viewer.is_running() and rclpy.ok():
            rclpy.spin_once(node, timeout_sec=sim.model.opt.timestep)   # wait ≤ 2 ms for a message
            sim.step_physics(start_time)                                 # catch physics up to wall clock
            viewer.sync()
```

{{< alert icon="lightbulb" >}}
**Why not `rclpy.spin(node)`?** The passive viewer's loop needs the main thread, and `spin()` would
block it forever. So ROS is **pumped** from inside the render loop, one `spin_once` per iteration.
{{< /alert >}}

- **`timeout_sec = timestep`.** A zero timeout lets the loop spin as fast as Python allows; waiting up
  to one physics step (2 ms) when idle still reacts within ~2 ms. Even so, the main thread measured
  **~90% of one core**, because every iteration also steps and syncs — capping the loop at display rate
  is a cheap improvement.
- **Real-time stepping.** `mj_step` runs until `data.time` catches up with wall-clock time, so the
  simulation runs at 1× regardless of loop rate.
- **The callback only writes `data.ctrl`** — no stepping, no rendering — so it takes microseconds.
- **The last command holds.** If the publisher dies, the motors keep their last forces, and with the
  switch-like physics the hand freezes in its last pose. A watchdog is on the roadmap.

## Observe it, poke it

From inside either container — or any LAN machine with ROS 2 Jazzy and `ROS_DOMAIN_ID=42`:

```bash
docker compose exec mujoco_twin bash
source /opt/ros/jazzy/setup.bash

ros2 node list                              # /vision_tracker_node  /mujoco_twin_node
ros2 topic info -v /hand/target_flexions    # endpoints and QoS
ros2 topic hz /hand/target_flexions         # 30.0 Hz alone · 10.7 Hz with the viewer open
ros2 topic echo /hand/target_flexions

# drive the twin with no camera: index and thumb closed
ros2 topic pub -r 10 /hand/target_flexions sensor_msgs/msg/JointState \
  "{name: [thumb, index, middle, ring, pinky], position: [1.0, 1.0, 0.0, 0.0, 0.0]}"

# record a session to replay into the twin later
ros2 bag record /hand/target_flexions
```

`ros2 topic pub` plus `docker compose up mujoco_twin` is the fastest way to develop the simulation
side without a webcam.
