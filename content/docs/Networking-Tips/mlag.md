---
title: "Multi Chassis Link Aggregation Basics"
bookCollapseSection: true
weight: 10
---

# Intro
## Link Aggregation Basics

Link aggregation is an ancient technology that allows you to bond multiple parallel links into a single virtual link (link aggregation group – LAG). With parallel links being replaced by a single virtual link, STP detects no loops and all the physical links can be fully utilized.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/s200-LAG_Basic.png)

I was also told that link aggregation makes your bridged network more stable. Every link state change triggers a Topology Change Notification in spanning tree, potentially sending a large part of your network through the STP listening-learning-forwarding state diagram. A link failure of a LAG member does not change the state of the aggregation group (unless it was the last working link in the group), and since STP rides on top of aggregated interfaces, it does not react to the failure.

## Multi-Chassis Link Aggregation(MLAG)
Imagine you could pretend two physical boxes use a single control plane and coordinated switching fabrics. The links terminated on two physical boxes would seem to terminate within the same control plane and you could aggregate them using LACP. Welcome to the wonderful world of Multi-Chassis Link Aggregation (MLAG).

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/s200-MCLA.png)
![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/mlag-1614767578-gTnizo8yuG.png)

MLAG nicely solves two problems:

* When used in the network core, multiple links appear as a single link. No bandwidth is wasted due to STP loop prevention while you still retain almost full redundancy – just make sure you always read the smallprint to understand what happens when one of the two switches in the MLAG pair fails.
* When used between a server and a pair of top-of-rack switches, it allows the server to use the full aggregate bandwidth while still retaining resiliency against a link or switch failure.

## MC-LAG (Multi-Chassis Link Aggregation Group)

### Overview

MC-LAG achieves similar redundancy and load balancing but is used primarily in service provider networks. Unlike MLAG, MC-LAG often uses a **control plane protocol (e.g., ICCP - Inter-Chassis Control Protocol)** to sync state information.

### Architecture

- Uses a **control protocol (ICCP)** to synchronize state between switches.
- Traffic forwarding decisions are synchronized between nodes.
- Unlike MLAG, which is mostly layer 2, MC-LAG may operate at **Layer 3 for better scaling**.

### Vendor-Specific Implementations

| Vendor | MC-LAG Implementation | Key Features |
| --- | --- | --- |
| **Juniper** | MC-LAG | Uses ICCP for control-plane synchronization |
| **Nokia** | MC-LAG | Supports both L2 and L3 redundancy |
| **Cisco (SP networks)** | MC-LAG | Used in service provider routers (ASR, NCS series) |
| **Huawei** | MC-LAG | Provides link aggregation for core networks |

## Key Technical Differences: MLAG vs MC-LAG

| Feature | **MLAG** | **MC-LAG** |
| --- | --- | --- |
| **Primary Use Case** | Data center networks | Service provider networks |
| **Interoperability** | Vendor-specific (Cisco vPC, Arista MLAG) | More interoperable (Juniper, Nokia, Cisco SP) |
| **Control Plane** | No dedicated protocol; uses peer links | Uses ICCP or proprietary control protocols |
| **Layer of Operation** | Layer 2 primarily | Layer 2 & Layer 3 |
| **Failure Handling** | Uses peer-link recovery methods | Uses ICCP for state synchronization |
| **Complexity** | Easier to configure | More complex due to control-plane sync |
| **Scalability** | Works well for small to medium-sized networks | Scales better in larger SP environments |

## Link Aggregation Control Protocol (LACP)
LACP (Link Aggregation Control Protocol):  a subcomponent of IEEE 802.3ad standard, provides a method to control the bundling of several physical ports together to form a single logical channel. LACP allows a network device to negotiate an automatic bundling of links by sending LACP packets to the peer.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/lacp-1614767595-ZGZFrYPm0H.png)

**References:**
* https://blog.ipspace.net/2010/10/multi-chassis-link-aggregation-basics.html
* https://community.fs.com/article/mlag-vs-stacking-vs-lacp.html
