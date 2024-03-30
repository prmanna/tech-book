---
title: "Spine-leaf Architecture Basics"
bookCollapseSection: true
weight: 10
---

# Intro
## What Is Spine-leaf Architecture?
The spine-leaf architecture consists of only two layers of switches: spine and leaf switches. The spine layer consists of switches that perform routing and work as the core of the network. The leaf layer involves access switches that connect to servers, storage devices, and other end-users. This structure helps data center networks reduce hop count and reduce network latency.

In the spine-leaf architecture, each leaf switch is connected to each spine switch. With this design, any server can communicate with any other server, and there is no more than one interconnected switch path between any two leaf switches.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/577kf_1HGFbK.jpg)

## Why Use Spine-leaf Architecture?
The spine-leaf architecture has become a popular data center architecture, bringing many advantages to the [data center](https://community.fs.com/blog/what-is-a-data-center.html), such as scalability, network performance, etc. The benefits of spine-leaf architecture in modern networks are summarized here in five points.

**Increased redundancy:** The spine-leaf architecture connects the servers with the core network, and has higher flexibility in hyper-scale data centers. In this case, the leaf switch can be deployed as a bridge between the server and the core network. Each leaf switch connects to all spine switches, which creates a large non-blocking fabric, increasing the level of redundancy and reducing traffic bottlenecks.

**Increased bandwidth:** The spine-leaf architecture can effectively avoid traffic congestion by applying protocols or techniques such as transparent interconnection of multiple links (TRILL) and shortest path bridging (SPB). The spine-leaf architecture can be Layer 2 or Layer 3, so uplinks can be added to the spine switch to expand inter-layer bandwidth and reduce oversubscription to secure network stability.

**Improve scalability:** The spine-leaf architecture has multiple links that can carry traffic. The addition of switches will improve scalability and help enterprises to expand their business later.

**Reduced expenses:** A spine-leaf architecture increases the number of connections each switch can handle, so data centers require fewer devices to deploy. Many data center networks employ spine-leaf architecture to minimize costs.

**Minimal latency and congestion:** By limiting the maximum number of hops to two between any source and destination nodes, we establish a more direct traffic path, enhancing overall performance and mitigating bottlenecks. The only exception is when the destination is on the same leaf switch.

### Spine-leaf vs. Traditional Three-Tier Architecture
The main difference between spine-leaf architecture and 3-tier architecture lies in the number of network layers, and the traffic they transform is north-south or east-west traffic.

As shown in the following figure, the traditional three-tier network architecture consists of three layers: core, aggregation and access. The access switches are connected to servers and storage devices, the aggregation layer aggregates the access layer traffic, provides redundant connections at the access layer, and the core layer provides network transmission. But this three-layer topology is usually designed for north-south traffic and uses the STP protocol, supporting up to 100 switches. In the case of continuous expansion of network data, this will inevitably result in port blockage and limited scalability.

The spine-leaf architecture is to add east-west traffic parallelism to the north-south network architecture of the backbone, fundamentally solving the bottleneck problem of the traditional three-tier network architecture. It increases the exchange layer under the access layer, and the data transmission between two nodes is completed directly at this layer, thereby diverting backbone network transmission. Compared with traditional three-tier architecture, the spine-leaf architecture provides a connection through the spine with a single hop between leaves, minimizing any latency and bottle necks. In spine-leaf architectures, the switch configuration is fixed so that no network changes are required for a dynamic server environment.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/Meiy5_2HM6Ed.jpg)

## How to Design Spine-leaf Architecture?
Before designing a spine-leaf architecture, you need to figure out some important and relevant considerations, especially the oversubscription rate and the size of the spine switch. Surely, we have also given a detailed example for your reference.

### Design Considerations of Spine-leaf Architecture

**Oversubscription rate:** It is the contention rate when all devices are sending traffic at the same time. It can be measured in the north/south direction (traffic entering/leaving the data center) and in the east/west direction (traffic between devices within the data center). The most appropriate oversubscription ratio for modern network architectures is 3:1 or less, which is measured and delineated as a ratio between upstream bandwidth (to backbone switches) and downstream capacity (to servers/storage).

For example, a leaf switch has 48 x 10G ports for a total of 480Gb/s of port capacity. If you connect 4 x 40G uplink ports from each leaf switch to a 40G spine switch, it will have 160Gb/s of uplink capacity. The ratio is 480:160, or 3:1. However, data center uplinks are typically 40G or 100G and can be migrated over time from a starting point of 40G (Nx 40G) to 100G (Nx 100G). It is important to note that the uplink should always run faster than the downlink to not have port link blockage.

**Leaf and spine sizing:** The maximum number of leaf switches in the topology is determined by the port density of the spine switches. And the number of spine switches will be governed by the combination of the required throughput between the leaf switches, the number of redundant/ECMP (equivalent multipath) paths, and their port density. So the number of spine-leaf switches and port density need to be taken into account to prevent network problems.

**Layer 2 or Layer 3 design:** A two-tier spine-leaf fabric can be built at either Layer 2 (configuring VLANs) or Layer 3 (subnetting). Layer 2 designs need to provide maximum flexibility, allowing VLANs to span anywhere and MAC addresses to migrate anywhere. Layer 3 designs need to provide the fastest convergence times and maximum scale with fan-out ECMP supporting up to 32 or more active spine switches.

**References:**
* https://community.fs.com/article/leaf-spine-with-fs-com-switches.html
