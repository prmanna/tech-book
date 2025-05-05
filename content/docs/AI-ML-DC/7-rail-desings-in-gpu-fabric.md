---
title: "Rail Designs in GPU Fabric"
bookCollapseSection: true
weight: 700
---

### Rail Designs in GPU Fabric

 When building a scalable, resilient GPU network fabric, the design of the rail layer, the portion of the topology that interconnects GPU servers via Top-of-Rack (ToR) switches, plays a critical role. This section explores three different models: Multi-rail-per-switch, Dual-rail-per-switch, and Single-rail-per-switch. All three support dual-NIC-per-GPU designs, allowing each GPU to connect redundantly to two separate switches, thereby removing the Rail switch as a single point of failure.

### Multi-Rail-per-Switch

In this model, multiple small subnets and VLANs are configured per switch, with each logical rail mapped to a subset of physical interfaces. For example, a single 48-port switch might host four or eight logical rails using distinct Layer 2 and Layer 3 domains. Because all logical rails share the same physical device, isolation is logical. As a result, a hardware or software failure in the switch can impact all rails and their associated GPUs, creating a large failure domain.

This model is not part of NVIDIA’s validated Scalable Unit (SU) architecture but may suit test environments, development clusters, or small-scale GPU fabrics where hardware cost efficiency is a higher priority than strict fault isolation. From a CapEx perspective, multi-rail-per-switch is the most economical, requiring fewer switches. 

Figure 13-10 illustrates the multi-rail-per-switch architecture, where each rail is implemented as a separate VLAN-subnet pair mapped to a subset of switch ports. In the figure, interfaces 1–4 are assigned to subnet 10.0.1.0/28 and VLAN 101, while interfaces 5–8 are mapped to subnet 10.0.2.0/28 and VLAN 102. Each VLAN maintains its own MAC address table, learning GPU NIC MACs through ingress traffic. Although not shown in the figure, the Rail switch acts as the default gateway for all eight VLANs.

The figure also illustrates the BGP process when a Clos architecture with a spine layer is used to connect rail switches. All directly connected subnets are installed into the local Routing Information Base (RIB) as connected routes. These routes are then imported into the BGP Loc-RIB. Next, the routes pass through the BGP output policy engine, where they are aggregated into a single summary route: 10.0.1.0/24. This aggregate is placed into the BGP Adj-RIB-Out. When the BGP Update message is sent to a peer, the NEXT\_HOP attribute is set accordingly.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/rail-desings/image1-1.png)

**Figure 13-10:** _Multi-Rail per Switch._

#### Dual-Rail-per-Switch

While dual-rail-per-switch improves manageability and is easier to scale, it shares the same limitation: both logical rails reside within a single physical switch, so the failure domain remains large. A single switch failure or misconfiguration affects both rails and all associated GPUs. 

This design resembles the dual-rail concept used in scalable AI clusters, but NVIDIA’s SU approach calls for two separate physical switches per rail, which provides full physical isolation. Dual-rail-per-switch hits a middle ground in terms of CapEx and OpEx: fewer switches are required than in the single-rail model, and operational complexity is reduced compared to multi-rail. It’s often a good choice for intermediate-scale environments where some fault tolerance and cost control must be balanced. 
  
Figure 13-11 illustrates a dual-rail-per-switch design, where the switch interfaces are divided evenly between two separate rails. Rail 1 uses interfaces 1 through 16 and is assigned to subnet 10.0.1.0/25 (VLAN 101). Rail 2 uses interfaces 17 through 32 and is assigned to subnet 10.0.128.0/25 (VLAN 102). Each VLAN has its own MAC address table, and the rail switch serves as the default gateway for both. The individual /25 subnets are redistributed into the BGP process and summarized as 10.0.1.0/24 for advertisement toward the spine layer.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/rail-desings/image1-2.png)

**Figure 13-11:** _Dual-Rail Switch._

#### Single-Rail-per-Switch

This model offers the highest level of physical isolation. Each switch forms a single rail, serving its connected GPU servers through one subnet and one VLAN. No logical separation is needed, as each rail is entirely independent in hardware. As a result, a switch failure affects only the GPU servers attached to that specific rail, yielding a small, predictable failure domain.

The design closely aligns with NVIDIA’s Scalable Unit (SU) architecture, in which each rack or rack group includes its own rail switch, and horizontal scaling is achieved by repeating modular, self-contained units.

While this model demands the highest CapEx, due to the one-to-one mapping between switches and rails, it offers major operational advantages. Configuration is simpler, troubleshooting is faster, and the risk of cascading faults is minimized. There is no need for route summarization, or custom BGP redistribution logic. Over time, these benefits help drive down OpEx, particularly in large-scale or mission-critical GPU clusters.

To ensure optimal hardware utilization, it is important to align the number of GPU servers per rack with the switch’s port capacity. Otherwise, underutilized ports can lead to inefficiencies in infrastructure cost and resource planning.

Figure 13-12 illustrates a simplified single-rail-per-switch topology. All interfaces from 1 to 32 operate within a single rail, configured with subnet 10.0.1.0/24 and VLAN 101. The rail switch serves as the default gateway, and because the full /24 subnet is used without subnetting, route summarization is not needed.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/rail-desings/image1-3.png)

**Figure 13-12:** _Single-Rail Switch._

### AI Fabric Architecture Conclusion

Figure 13-13 illustrates one way to describe the overall architecture of an AI Fabric. It is divided into three domains. The first domain, called the Segment, includes GPU hosts and Rail switches. The second domain, the Pod, aggregates multiple segments using Spine switches. In cases where NCCL builds a topology where cross-rail inter-host traffic is first copied to the local GPU memory (located on the destination rail) and then sent over the GPU NIC to the remote GPU via the correct Rail switch, a Pod architecture with Spine switches may not be necessary. The third domain, multi-Pod, interconnects multiple pods using Super Spine switches, enabling large-scale AI Fabric deployments. Figure 13-10 also depicts global settings and properties shared across the AI Fabric backend network.

#### Segment: GPU I/O Topology and Rail Switch Fabric Profile

GPU I/O Topology: Each GPU connects to the network through a NIC. You can either dedicate a NIC to each GPU or share one NIC among multiple GPUs. NICs may have single, dual, or quad ports and support speeds such as 100, 200, or 400 Gbps. The interconnect type can be InfiniBand, RoCEv2, or NVLink. A segment typically includes multiple hosts.

Rail Switch Fabric Profile: Rail switches connect directly to GPU hosts. Each rail handles a group of NIC ports. You can map rails one-to-one to switches for physical isolation or map multiple rails per switch for logical isolation. In the latter case, two or more rails can be mapped per switch depending on performance and capacity requirements. Rail switches are responsible for ingress packet classification and for mapping RoCEv2 traffic to the correct queues. 

  

#### Pod: Spine Switch Profile:
Spine switches aggregate multiple Rail switches, forming a Pod that consists of n segments. Spine switches enable cross-rail communication between GPUs. They use high-density, high-speed ports. When the Spine layer is used, the result is a 2-tier, 3-stage architecture.

#### Multi-Pod: Super Spine Switch Profile
Super Spine switches provide inter-Pod connectivity. They are built with very high port density to support all connected Spine switches. When the Super Spine layer is used, the architecture becomes a 3-tier, 5-stage fabric.

#### Global AI Fabric Profile
All layers are governed by the Global AI Fabric Profile. This profile defines the control plane (eBGP, iBGP, BGP EVPN), the data plane (Ethernet, VXLAN), Layer 3 ECMP strategies (flow-based, flowlet-based, or per-packet), congestion control mechanisms (ECN marking, PFC), inter-switch link monitoring (BFD), and global MTU settings.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/rail-desings/image1-4.png)

**Figure 13-13:** _AI fabric Architecture Description._

**References:**
* https://nwktimes.blogspot.com/2025/05/ai-for-network-engineers-rail-desings.html
