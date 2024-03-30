---
title: "Multi Chassis Link Aggregation Basics"
bookCollapseSection: true
weight: 10
---

# Intro
## Link Aggregation Basics

Link aggregation is an ancient technology that allows you to bond multiple parallel links into a single virtual link (link aggregation group – LAG). With parallel links being replaced by a single virtual link, STP detects no loops and all the physical links can be fully utilized.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/s200-LAG_Basic.png)
I was also told that link aggregation makes your bridged network more stable2. Every link state change triggers a Topology Change Notification in spanning tree, potentially sending a large part of your network through the STP listening-learning-forwarding state diagram. A link failure of a LAG member does not change the state of the aggregation group (unless it was the last working link in the group), and since STP rides on top of aggregated interfaces, it does not react to the failure.

## Multi-Chassis Link Aggregation
Imagine you could pretend two physical boxes use a single control plane and coordinated switching fabrics. The links terminated on two physical boxes would seem to terminate within the same control plane and you could aggregate them using LACP. Welcome to the wonderful world of Multi-Chassis Link Aggregation (MLAG).

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/s200-LAG_Basic.png)

MLAG nicely solves two problems:

* When used in the network core, multiple links appear as a single link. No bandwidth is wasted due to STP loop prevention while you still retain almost full redundancy – just make sure you always read the smallprint to understand what happens when one of the two switches in the MLAG pair fails.
* When used between a server and a pair of top-of-rack switches, it allows the server to use the full aggregate bandwidth while still retaining resiliency against a link or switch failure.

**References:**
* https://www.grotto-networking.com/BBQoS.html
