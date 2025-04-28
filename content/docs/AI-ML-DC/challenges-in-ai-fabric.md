---
title: "Challenges in AI Fabric Design"
bookCollapseSection: true
weight: 10
---

## Intro
Figure 10-1 illustrates a simple distributed GPU cluster consisting of three GPU hosts. Each host has two GPUs and a Network Interface Card (NIC) with two interfaces. Intra-host GPU communication uses high-speed NVLink interfaces, while inter-host communication takes place via NICs over slower PCIe buses.

GPU-0 on each host is connected to Rail Switch A through interface E1. GPU-1 uses interface E2 and connects to Rail Switch B. In this setup, inter-host communication between GPUs connected to the same rail passes through a single switch. However, communication between GPUs on different rails goes over three hops  Rail–Spine–Rail switches.

In Figure 10-1, we use a data parallelization strategy where a training dataset is split into six micro-batches, which are distributed across the GPUs. All GPUs use the shared feedforward neural network model and compute local model outputs. Next, each GPU calculates the model error and begins the backward pass to compute neuron-based gradients. These gradients indicate how much, and in which direction, the weight parameters should be adjusted to improve the training result (see Chapter 2 for details).

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/challenges-in-ai-fabric/image10-1.png)

**Figure 10-1:** _Rail-Optimized Topology._

### Egress Interface Congestions

After computing all gradients, each GPU stores the results in a local memory buffer and starts a communication phase where computed gradients are shared with other GPUs. During this process, the data (gradients) is being sent from one GPU and written to another GPU's memory (RDMA Write operation). RDMA is explained in detail in Chapter 9.

Once all gradients have been received, each GPU averages the results (AllReduce) and broadcasts the aggregated gradients to the other GPUs. This ensures that all GPUs update their model parameters (weights) using the same gradient values. The Backward pass process and gradient calculation are explained in Chapter 2.

Figure 10-2 illustrates the traffic generated during gradient synchronization from the perspective of GPU-0 on Host-2. Gradients from the local host’s GPU-1 are received via the high-speed NVLink interface, while gradients from GPUs in other hosts are transmitted over the backend switching fabric. In this example, all hosts are connected to Rail Switches using 200 Gbps fiber links. Since GPUs can communicate at line rate, gradient synchronization results in up to 800 Gbps of egress traffic toward interface E1 on Host-2, via Rail Switch A. This may cause congestion, and packet drops if the egress buffer on Rail Switch A or the ingress buffer on interface E1 is not deep enough to accommodate the queued packets.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/challenges-in-ai-fabric/image10-2.png)

**Figure 10-2:** _Congestion During Backward Pass._

### Single Point of Failure

The training process of a neural network is a long-running, iterative task where GPUs must communicate with each other. The frequency and pattern of this communication depend on the chosen parallelization strategy. For example, in data parallelism, communication occurs during the backward pass, where GPUs synchronize gradients. In contrast, model parallelism and pipeline parallelism involve communication even during the forward pass, as one GPU sends activation results to the next GPU holding the subsequent layer. It is important to understand that communication issues affecting even a single GPU can delay or interrupt the entire training process. This makes the AI fabric significantly more sensitive to single points of failure compared to traditional data center fabrics.

Figure 10-3 highlights several single points of failure that may occur in real-world environments. A host connection can become degraded or completely fail due to issues in the host, NIC, rail switch, transceiver, or connecting cable. Any of these failures can isolate a GPU. While this might not seem serious in large clusters with thousands of GPUs, as discussed in the previous section, even one isolated or failed GPU can halt the training process.

Problems with interfaces, transceivers, or cables in inter-switch links can cause congestion and delays. Similar issues arise if a spine switch is malfunctioning. These types of failures typically affect inter-rail traffic but not intra-rail communication. A failure in a rail switch can isolate all GPUs connected to that rail, creating a critical point of failure for a subset of the GPU cluster.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/challenges-in-ai-fabric/image10-3.png)

**Figure 10-3:** _Single-Point Failures._

### Head-of-Line Blocking 

In this example GPU clusters, NCCL (NVIDIA Collective Communications Library)  has built a topology where gradients are first sent from GPU-0 to GPU-1 over NVLink, and then forwarded from GPU-1 to other GPU-1s via Rail switch B.

However, this setup may lead to head-of-line blocking. This happens when GPU-1 is already busy sending its own gradients to the other GPUs, and now it also needs to forward GPU-0’s gradients. Since the PCIe and NIC bandwidth is limited, GPU-0’s traffic may need to wait in line behind GPU-1’s traffic. This queueing delay is called head-of-line blocking, and it can slow down the whole training process. The problem is more likely to happen when many GPUs rely on a single GPU or NIC for forwarding traffic to another rail. Even if only one GPU is overloaded, it can cause delays for others too.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/challenges-in-ai-fabric/image10-4.png)

**Figure 10-4:** _Head-of-Line Blocking._

### Hash-Polarization with ECMP

First, when two GPUs open a Queue Pair (QP) between each other, all gradient synchronization traffic is typically sent over that QP. From the network point of view, this looks like one large flow between the GPUs. In deep learning training, gradient data can be hundreds of megabytes or even gigabytes, depending on the model size. So, when it is sent over one QP, the network sees it as a single high-bandwidth flow. This kind of traffic is often called an elephant flow, because it can take a big share of the link bandwidth. This becomes a problem when multiple large flows are hashed to the same uplink or spine port. If that happens, one link can get overloaded while others remain underused. This is one of the reasons we see hash polarization and head-of-line blocking in AI clusters. Hash polarization is a condition where the load-balancing hash algorithm used in ECMP (Equal-Cost Multi-Path) forwarding results in uneven distribution of traffic across multiple available paths.

For example, in Figure 10-5, GPU-0 in Host-1 and GPU-0 in Host-2 both send traffic to GPU-1 at a rate of 200 Gbps. The ECMP hash function in Rail Switch A selects the link to Spine 1 for both flows. This leads to a situation where one spine link carries 400 Gbps of traffic, while the other remains idle. This is a serious problem in AI clusters because training jobs generate large volumes of east-west traffic between GPUs, often at line rate. When traffic is unevenly distributed due to hash polarization, some links become congested while others are idle. This causes packet delays and retransmissions, which can slow down gradient synchronization and reduce the overall training speed. In large-scale clusters, even a small imbalance can have a noticeable impact on job completion time and resource efficiency.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/challenges-in-ai-fabric/image10-5.png)

**Figure 10-5:** _ECMP Hash-Polarization._

The rest of this book focuses on how these problems can be mitigated or even fully avoided. We will look at design choices, transport optimizations, network-aware scheduling, and alternative topologies that help improve the robustness and efficiency of the AI fabric.

**References:**
* https://nwktimes.blogspot.com/2025/04/ai-for-network-engneers-challenges-in.html
