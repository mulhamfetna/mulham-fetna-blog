---
title: "ROS 2 across Docker containers: shared network, discovery and Fast DDS shared memory"
slug: "ros2-docker-network-shared-memory"
date: 2026-09-16
draft: false
description: "How two ROS 2 Jazzy Docker containers discover each other and share data: why the default bridge network breaks DDS multicast discovery, network_mode host, ROS_DOMAIN_ID and ROS_AUTOMATIC_DISCOVERY_RANGE for LAN visibility, and ipc/pid host for Fast DDS shared-memory transport — with evidence and hardening options."
keywords: ["ROS 2 Docker network", "ROS 2 docker discovery not working", "Fast DDS shared memory Docker", "ipc host docker ROS 2", "ROS_AUTOMATIC_DISCOVERY_RANGE", "ROS_DOMAIN_ID ports", "DDS multicast docker bridge", "SROS 2 security"]
tags: ["ros2", "docker", "networking", "robotics"]
categories: ["Projects"]
series: ["ROS 2 Tendon-Driven Hand MuJoCo Twin"]
series_order: 17
showDate: true
showAuthor: true
showTableOfContents: true
heroStyle: "background"
---

{{< katex >}}

{{< lead >}}
"The topic shows up in `ros2 topic list`, but `echo` prints nothing" — the classic ROS-in-Docker
symptom. Here's why it happens, how two containers in this project share a network and a block of
memory instead, and what that costs.
{{< /lead >}}

## How ROS 2 nodes find each other

ROS 2 has no master. Nodes discover each other through **DDS** — in Jazzy, **eProsima Fast DDS** by
default — using the SPDP protocol:

1. Each participant announces itself over **UDP multicast** (239.255.0.1) on ports derived from the
   domain ID \(d\): discovery multicast on \(7400 + 250d\), unicast on \(7410 + 250d + 2p\) for
   participant \(p\). For **domain 42**: UDP **17900** and **17910+**.
2. Peers exchange their topic endpoints; matching publishers and subscribers connect.
3. Data flows over the best **transport** both support — shared memory when they share a host and
   `/dev/shm`, UDP otherwise.

## Why Docker's default network breaks it

Compose attaches services to a **bridge network** — a private NATed subnet. Multicast isn't reliably
routed across it, and machines on your LAN can't reach container IPs at all.

**`network_mode: host`** removes the network namespace: the containers use the host's interfaces
directly, and discovery behaves as if ROS were installed natively.

| Option | Discovery between containers | LAN visibility | Network isolation |
|---|---|---|---|
| Bridge (default) | unreliable (multicast) | ✘ | good |
| Bridge + static peers / Discovery Server | ✔ with config | needs port mapping | good |
| **`network_mode: host`** — this project | **✔** | **✔** | none |

## Scope: domain ID and discovery range

```yaml
ROS_DOMAIN_ID: 42
ROS_AUTOMATIC_DISCOVERY_RANGE: SUBNET
```

- **`ROS_DOMAIN_ID=42`** partitions the DDS space. Work on the default domain 0 stays separate; pick
  any free 0–101.
- **`ROS_AUTOMATIC_DISCOVERY_RANGE`** (Jazzy and later):

| Value | Who can discover the nodes |
|---|---|
| `LOCALHOST` | processes on this machine only |
| **`SUBNET`** — this project | any machine on the local subnet with the same domain ID |
| `OFF` | nobody automatically — use static peers |
| `SYSTEM_DEFAULT` | the middleware's own default |

**From another computer:**

```bash
export ROS_DOMAIN_ID=42
ros2 topic echo /hand/target_flexions
```

Nothing arrives? Same subnet, host firewall open for **UDP 17900–17930**, and no Wi-Fi client
isolation or multicast filtering. For networks that drop multicast, set static peers with
`ROS_STATIC_PEERS=<ip>` (Jazzy+) or use a wired link.

## Shared memory: the fast path between the containers

Fast DDS's **shared-memory transport** writes a message once into a segment under `/dev/shm`, and the
subscriber reads it in place — no UDP packets, no kernel network stack, no loopback copies. Across
**containers**, that needs sharing:

| Requirement | Compose setting | Without it |
|---|---|---|
| The same `/dev/shm` | **`ipc: host`** | each container gets a private 64 MB `/dev/shm`; segments are invisible to the peer and Fast DDS falls back to UDP |
| A shared process view | **`pid: host`** | *precaution, not tested without it* — Fast DDS tracks segment and port ownership with lock files to clean up after dead peers; sharing the PID namespace avoids cross-namespace ambiguity |
| Compatible users | both run as `root` | the subscriber can't open the publisher's segments |

### Evidence

During testing, a subscriber in `mujoco_twin` received a `JointState` published from `vision_tracker`.
While it ran, the vision container listed Fast DDS segments — including those of the subscriber living
in the *other* container:

```text
$ ls /dev/shm | grep fast        # inside vision_tracker
fastrtps_5a080c43d95a7658
fastrtps_5a080c43d95a7658_el
fastrtps_837f37c7f519d02a
fastrtps_837f37c7f519d02a_el
fastrtps_port17913
```

`fastrtps_port17913` follows the same RTPS port formula as UDP for domain 42:
\(7400 + 250 \times 42 + 11 + 2 \times 1 = 17913\), the unicast user-data port of participant 1. The
`*_el` files are segment locks. All of it is visible to both containers only because of `ipc: host`.

{{< mermaid >}}
flowchart LR
    subgraph VT["🐳 vision_tracker"]
        P["publisher"]
    end
    subgraph MT["🐳 mujoco_twin"]
        S["subscriber"]
    end
    P -- "write once" --> SHM[("/dev/shm<br>fastrtps_* segments<br>ipc: host")]
    SHM -- "read in place" --> S
    P -. "discovery · UDP 17900+<br>network_mode: host" .- S
{{< /mermaid >}}

### Does it matter for five floats?

Not much — UDP loopback would carry five floats at 30 Hz fine. Shared memory earns its keep once you
publish **images**: streaming the annotated camera frame as `sensor_msgs/Image` is about 0.9 MB per
640×480 RGB frame, 30 times a second. The configuration is in place so that extension is free.

## What it costs, and how to harden it

| Choice | Cost | Production alternative |
|---|---|---|
| `network_mode: host` | no network isolation | dedicated VLAN; or bridge + Fast DDS Discovery Server / static peers |
| `ipc: host` | containers can read any host SHM segment | a shared named volume at `/dev/shm` for just these two services |
| `pid: host` | containers see every host process | as above, or accept UDP between containers |
| `SUBNET` discovery | anyone on the LAN on domain 42 can read — **and publish** — hand commands | `LOCALHOST` for demos; **SROS 2** (DDS Security) for authenticated, encrypted topics |

{{< alert icon="shield" >}}
**Anyone on your subnet with `ROS_DOMAIN_ID=42` can publish to `/hand/target_flexions` and move the
twin.** Harmless for a simulator and handy for teaching. Before that topic drives real servos, enable
SROS 2 or restrict discovery.
{{< /alert >}}
