---
title: "Congestion Avoidance in AI Fabric"
bookCollapseSection: true
weight: 10
---

## Congestion Avoidance in AI Fabric - Part I: Explicit Congestion Notification (ECN)

As explained in the preceding chapter, “Egress Interface Congestions,” both the Rail switch links to GPU servers and the inter-switch links can become congested during gradient synchronization. It is essential to implement congestion control mechanisms specifically designed for RDMA workloads in AI fabric back-end networks because congestion slows down the learning process and even a single packet loss may restart the whole training process.

This section begins by introducing Explicit Congestion Notification (ECN) and Priority-based Flow Control (PFC), two foundational technologies used in modern lossless Ethernet networks. ECN allows switches to mark packets, rather than dropping them, when congestion is detected, enabling endpoints to react proactively. PFC, on the other hand, offers per-priority flow control, which can pause selected traffic classes while allowing others to continue flowing.

Finally, we describe how Datacenter Quantized Congestion Notification (DCQCN) combines ECN and PFC to deliver a scalable and lossless transport mechanism for RoCEv2 traffic in AI clusters.

### GPU-to-GPU RDMA Write Without Congestion

The figure 11-1 illustrates a standard Remote Direct Memory Access (RDMA) Write operation between two GPUs. This example demonstrates how GPU-0 on Host-1 transfers local gradients (∇₁ and ∇₂) from memory to GPU-0 on Host-2. Both GPUs use RDMA-capable NICs connected to Rail Switch A via 200 Gbps uplinks.

The RDMA Write operation proceeds through the following seven steps:

1. To initiate the data transfer, GPU-0 on Host-1 submits a work request to its RDMA NIC over the PCIe bus over the pre-established Queue Pair 0x123456. 
2. The RDMA NIC encodes the request by inserting the OpCode (RDMA Write) and Queue Pair Number (0x123456) into the InfiniBand Transport Header (IBTH). It wraps the IBTH and RETH (not shown in the figure) headers with Ethernet, IP, UDP, and Ethernet headers. The NIC sets the DSCP value to 24 and the ECN bits to 10 (indicating ECN-capable transport) in the IP header's ToS octet. The DSCP value ensures that the switch can identify and prioritize RoCEv2 traffic. The destination UDP port is set to 4791 (not shown in the figure).
3. Upon receiving the packet on interface Ethernet1/24, the Rail switch classifies the traffic as RoCEv2 based on the DSCP value of 24.
4. The switch maps DSCP 24 to QoS-Group 3.
5. QoS Group 3 uses egress priority queue 3, which is configured with bandwidth allocation and congestion avoidance parameters (WRED Min, WRED Max, and Drop thresholds) optimized for RDMA traffic. 
6. The packet count on queue 3 does not exceed the WRED minimum threshold, so packets are forwarded without modification. 
7. The RDMA NIC on Host-2 receives the packet, strips off the Ethernet, IP, and UDP headers, and processes the RDMA headers. The payload is delivered directly into the memory of GPU-0 on Host-2 without CPU involvement, completing the RDMA Write operation.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/congestion-avoidance/image1-1.png)

**Figure 11-1:** _Overview of Remote DMA operation Under Normal Condition._

### Explicit Congestion Notification -ECN

Because gradient synchronization requires a lossless network service, it is essential to have a proactive congestion detection system that can respond before buffer overflows occur. This system must also include a signalling mechanism that allows the receiver to request the sender to reduce its transmission rate or temporarily pause traffic when necessary.

Data Center Quantized Congestion Notification (DCQCN) is a congestion control scheme designed for RoCEv2 that leverages both Explicit Congestion Notification (ECN) and Priority Flow Control (PFC) for active queue management. This section focuses on ECN.

In IPv4, the last two bits (bits 6 and 7) of the ToS (Type of Service) byte are reserved for ECN marking. With two bits, four ECN codepoints are defined:

00 – Not ECN-Capable Transport (Not-ECT)
01 – ECN-Capable Transport (ECT)
10 – ECN-Capable Transport (ECT)
11 – Congestion Experienced (CE)

Figure 11-2 illustrates how ECN is used to prevent packet drops when congestion occurs on egress interface Ethernet2/24. ECN operates based on two queue thresholds: WRED Minimum and WRED Maximum. When the queue depth exceeds the WRED Minimum but remains below the WRED Maximum, the switch begins to randomly mark forwarded packets with ECN 11 (Congestion Experienced). If the queue depth exceeds the WRED Maximum, all packets are marked with ECN 11. The Drop Threshold defines the upper limit beyond which packets are no longer marked but instead dropped.

In Figure 11-2, GPU-0 on Host-1 transfers gradient values from its local memory to the memory of GPU-0 on Host-2. Although not shown in the figure for simplicity, other GPUs connected to the same rail switch also participate in the synchronization process with GPU-0 on Host-2. Multiple simultaneous elephant flows towards GPU-0 reach the rail switch, causing egress queue 3 on interface Ethernet2/24 to exceed the WRED Maximum threshold. As a result, the switch begins marking outgoing packets with ECN 11 (step 6), while still forwarding them to the destination GPU. 

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/congestion-avoidance/image1-2.png)

**Figure 11-2:** _Congested Egress Interface – ECN Congestion Experienced._

After receiving a data packet marked with ECN 11, the destination RDMA NIC must inform the sender about network congestion. This is accomplished using a Congestion Notification Packet (CNP), which provides feedback at the RDMA transport layer. 

1. Upon detecting the ECN  11 in the IP header’s ToS bits of an incoming packet, the RDMA NIC on Host-2 generates a CNP message. It sets the OpCode in the IBTH header to 0x81, identifying the message as a CNP. Besides the FECN bit is set to 1, indicating that the NIC experienced congestion. The Queue Pair number used for the original memory copy (e.g., 0x123456) is reused, ensuring the feedback reaches the correct sender-side transport context. In the IP header, the DSCP field is set to 48, allowing switches to distinguish the CNP from standard RoCEv2 data traffic.
2. When the CNP reaches the Rail switch (interface Ethernet2/24), it is classified based on DSCP 48, which is associated with CNP traffic in the QoS configuration.
3. DSCP 48 maps the packet to QoS group 7, which is reserved for congestion feedback signaling.
4. QoS group 7 is associated with strict-priority egress queue 7, ensuring that CNP packets are forwarded with the highest priority. This guarantees that congestion signals are not delayed behind other types of traffic.
5. The switch forwards the CNP to the originating NIC on Host-1. Because of the strict-priority handling, the feedback arrives quickly even during severe congestion.
6. Upon receiving the CNP, the sender-side RDMA NIC reduces its transmission rate for the affected Queue Pair by increasing inter-packet delay. This is achieved by holding outgoing packets longer in local buffers, effectively reducing traffic injection into the congested fabric.

As transmission rates decrease, the pressure on the egress queue at the Rail switch’s interface Ethernet1/14 (connected to Host-2) is gradually relieved. Buffer occupancy falls below the WRED Minimum Threshold, ending ECN marking. Once congestion is fully cleared, the RDMA NIC slowly ramp up its transmission rate. This gradual marking strategy helps prevent sudden traffic loss and gives the sender time to react by adjusting its sending rate before the buffer overflows.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/congestion-avoidance/image1-3.png)

**Figure 11-3:** _Receiving NIC Generates CNP - Sender NIC Delay Transmit._  
  
Next post describes DSCP-Based Priority Flow Control (PFC) use cases and operation.

## Congestion Avoidance in AI Fabric - Part II: Priority Flow Control (PFC)

Priority Flow Control (PFC) is a mechanism designed to prevent packet loss during network congestion by pausing traffic selectively based on priority levels. While the original IEEE 802.1Qbb standard operates at Layer 2, using the Priority Code Point (PCP) field in Ethernet headers, AI Fabrics rely on Layer 3 forwarding, where traditional Layer 2-based PFC is no longer applicable. To extend lossless behavior across routed (Layer 3) networks, DSCP-based PFC is used.

In DSCP-based PFC, the Differentiated Services Code Point (DSCP) field in the IP header identifies the traffic class or priority. Switches map specific DSCP values to internal traffic classes and queues. If congestion occurs on an ingress interface and a particular priority queue fills beyond a threshold, the switch can send a PFC pause frame back to the sender switch, instructing it to temporarily stop sending traffic of that class—just as in Layer 2 PFC, but now triggered based on Layer 3 classifications.

This behavior differs from Explicit Congestion Notification (ECN), which operates at Layer 3 as well but signals congestion by marking packets instead of stopping traffic. ECN acts on the egress port, informing the receiver to notify the sender to reduce the transmission rate over time. In contrast, PFC acts immediately at the ingress port, pausing traffic flow in real time to avoid buffer overflows and packet drops.

PFC relies on two thresholds to control flow: xOFF and xON. The xOFF threshold defines the point at which the switch generates a pause frame when a priority queue reaches a congested state. Once triggered, upstream devices halt transmission of that traffic class. The switch continuously monitors its buffer occupancy, and when the level drops below the xON threshold, it sends a PFC frame with a Quanta value of 0 for the affected priority. This signals the upstream device that it can resume transmission for that specific priority queue.

A key requirement for PFC to function correctly is the provisioning of buffer headroom. The switch must reserve enough buffer space per priority class to accommodate in-flight traffic while the pause frame propagates to the sender and takes effect.

DSCP-based PFC enables lossless packet delivery over routed networks, which is especially important for technologies like RoCEv2 (RDMA over Converged Ethernet v2), where even minimal packet loss can cause significant performance degradation.

### DSCP-Based PFC Process over a Layer 3 Routed Interface (Example Scenario)

This example illustrates how DSCP-based Priority Flow Control (PFC) operates across a routed Layer 3 fabric during congestion. We walk through a four-step process, beginning with buffer overflow and ending with traffic pausing on the correct priority queue.

#### Step 1: Buffer Overflow on Rail Switch C (Egress to GPU-3, Host 3)

In a GPU cluster, multiple GPUs are sending high-throughput RDMA traffic to GPU on Host-3. In Figure 11-4 Rail Switch C is responsible for forwarding traffic toward GPU-3. The egress interface on Switch C (E12/24) that connects to GPU-3 becomes congested. Due to the overflow of egress queue 3, packets from ingress queue 3 on interface E3/24 cannot be placed into egress queue 3.

#### Step 2: xOFF Threshold Exceeded

Priority queue 3  of has two configured thresholds:

* xOFF threshold: Triggers a pause when buffer usage exceeds this level. 
* xON threshold: Triggers a resume when the buffer has drained sufficiently.

Once priority queue 3 on ingress interface E3/24 exceeds its xOFF threshold, the switch takes immediate action to prevent packet loss by generating a PFC pause message targeted at the sender. The sender in this case is Spine Switch 1, which is sending traffic to Rail Switch C, over interface E3/24, for delivery to GPU-3.

#### Step 3: Generating a PFC Pause Frame (MAC Control Frame)
To pause the sender, Rail Switch C generates an Ethernet MAC Control frame with:

* **Ethertype  0x8808**: This indicates a MAC Control frame, used for pause-related operations (standardized in IEEE 802.3x). Inside this frame, a PFC opcode (0x0101) specifies it's a Priority-based Pause (PFC) message.
* **Class Enable Vector (CEV)**: This 8-bit field indicates which priority queues should be paused. Each bit corresponds to one of the 8 possible traffic classes (0–7). For example, if bit 3  is set to 1, it tells the sender to pause traffic for priority queue 3 only, while all other bits remain 0. In our case, the CEV is 0x1000. Note that the right-most bit represents queue 0.
* **Quanta Field(s)**: For each enabled priority (bit set to 1), a corresponding quanta value is specified. This value defines the duration of the pause, measured in units of 512 bit times.

For a 400 Gbps interface:
  
* 1 bit time = 1 / 400,000,000,000 seconds ≈ 2.5 picoseconds
* 1 quanta = 512 × 2.5 ps = 1.28 nanoseconds
* If the pause quanta is set to maximum value 0xFFFF (65535), the pause duration is roughly 83.9 microseconds.

This pause frame is sent back to the sender Spine Switch 1. Since the DSCP-based classification maps back to priority queue 3, and the switches share the same mapping, Spine Switch 1 will interpret this correctly.

#### Step 4: Spine Switch 1 Pauses Transmission on Priority Queue 3

Upon receiving the PFC frame on its ingress interface E3/24 connected to Rail Switch C, Spine Switch 1 examines the class enable vector.

* Since bit 3 is set, the switch knows to pause transmission of all frames mapped to priority queue 3 (DSCP value 24 in our example) on egress interface E3/24.
  
Traffic for other priority queues continues unaffected. 

Spine Switch 1 holds off transmission of priority 3 traffic until it receives a subsequent PFC frame with quanta = 0, indicating “resume,” or a pause duration timeout occurs, after which the switch resumes sending unless another pause is received.
  
![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/congestion-avoidance/image2-1.png)

**Figure 11-4:** _Priority Flow Control – Pause Frame._

The following example shows how Priority Flow Control (PFC) events can cascade upstream when congestion persists in a routed Layer 3 fabric. This scenario builds on the earlier case, where Spine Switch 1 paused traffic to Rail Switch C. Now, we observe how that pause affects traffic originating from Rail Switches A and B.

#### Step 5: Congestion on Spine Switch 1 Egress Queue to Rail Switch C

As described in the previous figure, Spine Switch 1 received a PFC frame from Rail Switch C and responded by pausing traffic on priority queue 3 on its egress interface E3/24 (towards Rail Switch C). Because this interface is no longer sending traffic, frames destined for GPU-3 via Rail Switch C begin to accumulate in Spine Switch 1’s egress queue 3. This build-up causes backpressure that impacts the ingress side of the switch.

#### Step 6: xOFF Threshold Exceeded on Spine Switch 1 Ingress Interfaces

Spine Switch 1 receives incoming traffic from Rail Switch A (interface E1/24) and Rail Switch B (interface E2/24). Both switches are sending traffic mapped to priority queue 3 (e.g., DSCP 24). As the egress queue to Rail Switch C becomes full and cannot drain, the corresponding ingress buffers on interfaces E1/24 and E2/24 also begin to fill up, specifically for queue 3. Eventually, the xOFF thresholds on both ingress interfaces are exceeded, indicating that congestion is now impacting the reception of new packets on these ports.

#### Step 7: Spine Switch 1 Sends PFC Pause Frames to Rail Switch A and B

To avoid dropping packets due to ingress buffer overflow, Spine Switch 1 generates PFC MAC Control frames on both E1/24 and E2/24. The class enable vector has bit 3 set, instructing the sender to pause traffic corresponding to priority queue 3. A suitable quanta value is included to define the pause duration. These control frames travel back to Rail Switch A and Rail Switch B respectively.

#### Step 8: Rail Switches A and B Pause Queue 3 Traffic to Spine Switch 1

Upon receiving the PFC frames, both Rail Switch A and Rail Switch B interpret the class enable vector and pause all traffic mapped to priority queue 3 (e.g., DSCP 24),  still forwarding traffic on other priority queues unaffected. This marks the upstream propagation of congestion: a single bottleneck on the path to GPU-3 can trigger PFC reactions all the way back to multiple source switches.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/congestion-avoidance/image2-2.png)

Figure 11-5: Priority Flow Control – Cascading Effect.

#### Steps 9a – 14: Downstream Resume and Congestion Recovery

Figure 11-6 illustrates how the PFC-based congestion recovery process extends from Rail Switches A and B all the way to the GPU NICs, while simultaneously resolving the initial congestion at Rail Switch C.

As a result of the earlier PFC pause frames:

* Rail Switch A and Rail Switch B have paused sending priority queue 3 traffic to Spine Switch 1.
* In turn, Spine Switch 1 has paused its own egress traffic toward Rail Switch C on interface E3/24.
  
This pause allows queue 3 on Rail Switch C’s egress interface E12/24 (toward GPU-3) to drain, as no new traffic is arriving, and the GPU continues to consume incoming data.

Once the buffer utilization for priority queue 3 drops below the configured xON threshold, Rail Switch C initiates congestion recovery.

* It sends a MAC Control Frame (Ethertype 0x8808) back to Spine Switch 1.
* The class enable vector has bit 3 set (indicating priority queue 3).
* The quanta value is set to 0, signaling that it is now safe to resume transmission.

Upon receiving this resume message, Spine Switch 1 can begin sending traffic again on priority queue 3, restoring throughput toward GPU-3 and continuing the flow of RDMA traffic through the network. This recovery mechanism operates consistently across the entire AI fabric.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/congestion-avoidance/image2-3.png)

Figure 11-6: Priority Flow Control – PCIe Bus Congested: Cascading Effect.

### LLDP with DCBX

PFC negotiation is performed using the Link Layer Discovery Protocol (LLDP), which carries Data Center Bridging eXchange (DCBX) Type-Length-Value (TLV) structures. At the time of writing, DCBX exists in two versions: IEEE and CEE. The IEEE mode (defined in 802.1Qbb and 802.1Qaz) is standards-based and supported by most modern data center switches from various vendors. This mode is also known as DCBXv2. Some older Cisco Nexus models support only the Cisco/Converged Enhanced Ethernet (CEE) mode. Capture 11-1 shows the packet format of a standards-based IEEE DCBX TLV within an LLDP message.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/congestion-avoidance/image2-4.png)

Capture 11-1: PCF: LLDP with IEEE DBCXv2 TLV .

## Congestion Avoidance in AI Fabric - Part III: Data Center Quantized Congestion Notification (DCQCN)

Data Center Quantized Congestion Notification (DCQCN) is a hybrid congestion control method. DCQCN brings together both Priority Flow Control (PFC) and Explicit Congestion Notification (ECN) so that we can get high throughput, low latency, and lossless delivery across our AI fabric. In this approach, each mechanism plays a specific role in addressing different aspects of congestion, and together they create a robust flow-control system for RDMA traffic.

DCQCN tackles two main issues in large-scale RDMA networks:

1. Head-of-Line Blocking and Congestion Spreading: This is caused by PFC’s pause frames, which stop traffic across switches.
2. Throughput Reduction with ECN Alone: When the ECN feedback is too slow, packet loss may occur despite the rate adjustments.

DCQCN uses a two-tiered approach. It applies ECN early on to gently reduce the sending rate at the GPU NICs, and it uses PFC as a backup to quickly stop traffic on upstream switches (hop-by-hop) when congestion becomes severe.

### How DCQCN Combines ECN and PFC

DCQCN carefully combines Explicit Congestion Notification (ECN) and Priority Flow Control (PFC) in the right sequence:

**Early Action with ECN:** When congestion begins to build up, the switch uses WRED thresholds (minimum and maximum) to mark packets. This signals the sender to gradually reduce its transmission rate. As a result, the GPU NIC slows down, and traffic continues flowing—just at a reduced pace—without abrupt pauses.

**Backup Action with PFC:** If congestion worsens and the queue continues to grow, the buffer may reach the xOFF threshold. At this point, the switch sends PFC pause frames hop by hop to upstream devices. These devices respond by temporarily stopping traffic for that specific priority queue, helping prevent packet loss.

**Resuming Traffic:** Once the buffer has drained and the queue drops below the xON threshold, the switch sends a resume message (a PFC frame with a quanta value of 0). This tells the upstream device it can start sending traffic again.

### Why ECN Must Precede xOFF

It is very important that the ECN thresholds (WRED minimum and maximum) are used before the xOFF threshold is reached for three main reasons:

**Graceful Rate Adaptation:** Early ECN marking helps the GPU NIC (sender) reduce its transmission rate gradually. This smooth adjustment avoids sudden stops and leads to more stable traffic flows.

**Avoiding Unnecessary PFC Events:** If the sender adjusts its rate early with ECN feedback, the buffers are less likely to fill up to the xOFF level. This avoids the need for abrupt PFC pause frames that can cause head-of-line blocking and backpressure on the network.

**Maintaining Fabric Coordination:** With early ECN marking, the sender receives feedback before congestion becomes severe. While the ECN signal is not shared directly with other switches, the sender's rate adjustment helps reduce overall pressure on the network fabric.

### What Happens If xOFF Is Reached Before ECN Marking?

Imagine that the ingress queue on Spine Switch 1 (from Rail Switch A) fills rapidly without ECN marking:

**Sudden Pause:** The buffer may quickly hit the xOFF threshold and trigger an immediate PFC pause.

**Downstream Effects:** An abrupt stop in traffic from Rail Switch A leads to sudden backpressure. This can cause head-of-line blocking and disturb GPU communication, leading to performance jitter or instability at the application level.

**Oscillations:** When the queue finally drains and reaches the xON threshold, traffic resumes suddenly. This can cause recurring congestion and stop-and-go patterns that hurt overall performance.

By allowing ECN to mark packets early, the network gives the sender time to reduce its rate smoothly. This prevents abrupt stops and helps maintain a stable, efficient fabric.

Figure 11 recaps how the example DCQCN process works:

**Time t1:** (1) Traffic associated with priority queue 3 on Rail-1’s egress interface 1 crosses the WRED minimum threshold.

**Time t2:** (2) Rail-1 begins randomly marking ECN bits as 11 on packets destined for GPU-0 on the Host-3.

**Time t3:** (3) The RDMA NIC starts sending CNP messages to the sender GPU-1 on Host-1.

**Time t4:** (4) In response to the CNP message, the sending GPU-0 on Host-1 reduces its transmission rate by holding packets longer in its egress queue. (5) At the same time, egress queue 3 on Rail-1 remains congested. (6) Since packets cannot be forwarded from ingress interface 2 to egress interface 1’s queue 3, ingress interface 3 also becomes congested, eventually crossing the PFC xOFF threshold.

**Time t5:** (7) As a result, Rail-1 sends a PFC xOFF message to Spine-A over Inter-Switch Link 3. (8) In response, Spine-A halts forwarding traffic for the specified pause duration.

**Time t6:** (9) Due to the forwarding pause, the egress queue of interface 3 on Spine-A becomes congested, which in turn (10) causes congestion on its ingress interface 2.

**Time t7:** (11) The number of packets waiting in egress queue 3 on interface 1 of Rail-1 drops below the WRED minimum threshold. (12) This allows packets from the buffer of interface 3 to be forwarded.

**Time t8:** (13) The packet count on ingress interface 3 of Rail-1 falls below the PFC xON threshold, triggering the PFC resume/unpause message to Spine-A. (14) Spine-A resumes forwarding traffic to Rail-1.

After the PFC resume message is sent, Spine-A starts forwarding traffic again toward Rail-1. The congestion on Spine-A’s interface 3 gets cleared as packets leave the buffer. This also helps the ingress interface 2 on Spine-A to drain. On Rail-1, as interface 1 can now forward packets, queue 3 gets more room, and the flow to GPU-0 becomes smoother again.

The RDMA NIC on the sender GPU monitors the situation. Since there are no more CNP messages coming in, the GPU slowly increases its sending rate. At the same time, the ECN marking on Rail-1 stops, as queue lengths stay below the WRED threshold. Traffic flow returns to normal, and no more PFC pause messages are needed.

The whole system stabilizes, and data can move again without delay or packet loss.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/congestion-avoidance/image3-1.png)

Figure 11-7: DCQCN: ECN and PFC Interaction .

### DCQCN Configuration

Figure 11-8 shows the six steps to enable DCQCN on a switch. The figure assumes that the RDMA NIC marks RoCEv2 traffic with DSCP 24.

First, we classify the packets based on the DSCP value in the IPv4 header. Packets marked with DSCP 24 are identified as RoCEv2 packets, while packets marked with DSCP 48 are classified as CNP.

After classification, we add an internal QoS label to the packets to place them in the correct output queue. The mapping between internal QoS labels and queues is fixed and does not require configuration.

Next, we define the queue type, allocate bandwidth, and set ECN thresholds. After scheduling is configured, we enable PFC and set its threshold values. A common rule of thumb for the relationship between ECN and PFC thresholds is: xON < WRED Min < WRED Max < xOFF.

To apply these settings, we enable them at the system level. Finally, we apply the packet classification to the ingress interface and enable the PFC watchdog on the egress interface. Because PFC is a sub-TLV in the LLDP Data Unit (LLDPDU), both LLDP and PFC must be enabled on every inter-switch link.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ai-ml-dc/congestion-avoidance/image3-2.png)

**Figure 11-8:** _Applying DCQCN to Switch._

#### Step 1: Packet Classification

The classification configuration is used to identify different types of traffic based on their DSCP values. In our example we have one for RoCEv2 traffic and another for Congestion Notification Packets (CNP). The “class-map type qos match-any ROCEv2” line defines a class map named “ROCEv2” that matches any packet marked with DSCP value 24, which is commonly used for RDMA traffic. Similarly, the “class-map type qos match-any CNP” defines another class map named “CNP” that matches packets marked with DSCP value 48, typically used for congestion signaling in RDMA environments. These class maps serve as the foundation for downstream policies, enabling differentiated handling of traffic types. Note that the names “ROCEv2” and “CNP” are not system-reserved; they are simply user-defined labels that can be renamed, as long, as the references are consistent throughout the configuration.

```
class-map type qos match-any ROCEv2 
  match dscp 24
class-map type qos match-any CNP 
  match dscp 48
```

**Example 11-1:** _Classification._

#### Step 2: Internal QoS Label for Queueing

The marking configuration assigns internal QoS labels to packets that have already been classified. This is done using a policy map named QOS\_CLASSIFICATION, which refers to the previously defined class maps. Within this policy, packets that match the “ROCEv2” class are marked with qos-group 3, and those matching the “CNP” class are marked with  qos-group 7. Any other traffic that doesn't fit these two categories falls into the default class and is marked with qos-group 0. These QoS groups are internal identifiers that the switch uses in later stages for  queuing and scheduling, to decide how each packet should be treated. Just like class maps, the name of the policy map itself is user-defined and can be anything descriptive, provided it is correctly referenced in other parts of the configuration. 

```
policy-map type qos QOS_CLASSIFICATION 
  class ROCEv2
    set qos-group 3
  class CNP
    set qos-group 7
  class class-default
    set qos-group 0
```
**Example 11-2:** _Marking._

#### Step 3: Scheduling

The queuing configuration defines how traffic is scheduled and prioritized on the output interfaces, based on the internal QoS groups that were assigned earlier. This is handled by a policy map named “QOS\_EGRESS\_PORT,” which maps traffic to different hardware output queues. Each queue is identified by a class, such as c-out-8q-q7 (fixed names: 8q = eight queues, q7 = queue number 7). For example, queue 7 is configured with priority level 1, which gives it strict priority over all other traffic. Queue 3 is assigned bandwidth remaining percent 50, meaning that it is guaranteed half of the remaining bandwidth after strict-priority traffic has been serviced. In addition to bandwidth allocation, queue 3 includes congestion management features through the random-detect command. This enables Weighted Random Early Detection (WRED), a mechanism that helps avoid congestion by randomly mark  packets as queue depth increases. The minimum-threshold and maximum-threshold define the WRED minimum and maximum values (from 150 KB to 3000 KB) at which packets begin marked. The drop-probability 7 determines the likelihood of packet mark when the maximum threshold is reached, with higher numbers indicating higher marking rates. The weight 0 setting controls how queue size is averaged. A weight of 0 means use instantaneous queue depth (no averaging). Finally, ecn enables Explicit Congestion Notification, allowing network devices to signal congestion without dropping packets, without the ecn option switch drops packet based on WRED min/max values. The remaining queues are configured with either zero percent of remaining bandwidth, effectively disabling them for general use, or with a share of the remaining bandwidth. This queuing policy ensures that RoCEv2 traffic receives adequate resources with congestion feedback, while CNP messages always get through with strict priority.

```
policy-map type queuing QOS\_EGRESS\_PORT
  class type queuing c-out-8q-q6
    bandwidth remaining percent 0
  ...

  class type queuing c-out-8q-q3
    bandwidth remaining percent 50
    random-detect minimum-threshold 150 kbytes maximum-threshold 3000 kbytes drop-probability 7 weight 0 ecn
  ...

  class type queuing c-out-8q-q7
    priority level 1 
```

**Example 11-3:** _Queuing (Output Scheduling)._

#### Step 4: Enable PFC for Queue

The Network QoS configuration defines the low-level, hardware-based characteristics of traffic handling within the switch, such as enabling lossless behavior and setting the maximum transmission unit (MTU) size for each traffic class. In this example, the policy-map type network-qos qos\_network is used to configure how traffic is handled inside the switch fabric. Under this policy, the class type network-qos c-8q-nq3 is associated with pause pfc-cos 3, which enables Priority Flow Control (PFC) on Class of Service (CoS) 3. This is critical for RoCEv2 traffic, which depends on a lossless transport layer. The MTU is also defined here, with bytes (jumbo frame) set for class 3 traffic.

```
policy-map type network-qos qos\_network
  class type network-qos c-8q-nq3
   mtu 9216    
   pause pfc-cos 3   
```

**Example 11-4:** _Queuing (Output Scheduling)._

#### Priority Flow Control Watchdog

The Priority Flow Control (PFC) watchdog is a mechanism that protects the network from traffic deadlocks caused by stuck PFC pause frames. In RDMA environments like RoCEv2, PFC is used to create lossless classes of traffic by pausing traffic flows instead of dropping packets. However, if a device fails to release the pause or a misconfiguration causes PFC frames to persist, traffic in the affected class can become permanently blocked, leading to what is called a "head-of-line blocking" condition. To mitigate this risk, the priority-flow-control watch-dog-interval on command enables the PFC watchdog feature. When enabled, the switch monitors traffic in each PFC-enabled queue for signs of persistent pause conditions. If it detects that traffic has been paused for too long, indicating a potential deadlock, it can take corrective actions, such as generating logs, resetting internal counters, or even discarding paused traffic to restore flow. 

```
priority-flow-control watch-dog-interval on
```
  
**Example 11-5:** _Priority Flow Control (PFC) Watchdog._

#### Step 5: Bind and Apply QoS Settings

System-level QoS policies bind all the previously defined QoS components together and activate them across the switch. This is done using the system qos configuration block, which applies the appropriate policy maps globally. The service-policy type network-qos qos\_network command activates the network-qos policy defined earlier, ensuring that MTU sizes and PFC configurations are enforced across the switch fabric. The command service-policy type queuing output QOS\_EGRESS\_PORT applies the queuing policy at the output interface level, enabling priority queuing, bandwidth allocation, and congestion management as traffic exits the switch. These system-level bindings are essential because, without them, the individual QoS policies, classification, marking, queuing, and fabric-level configuration, would remain inactive. By applying the policies under system qos, the switch is instructed to treat traffic according to the rules and priorities defined in each policy map. This final step ensures end-to-end consistency in QoS behavior, from ingress classification to fabric transport and egress scheduling, providing a complete and operational quality-of-service framework tailored for latency-sensitive, lossless applications like RoCEv2.
  
```
system qos
  service-policy type network-qos qos\_network
  service-policy type queuing output QOS\_EGRESS\_PORT
```

**Example 11-6:** _Priority Flow Control (PFC) Watchdog._

Step 6: Interface-Level Configuration 

The interface-level configuration attaches the previously defined QoS policies and enables PFC-specific features for a given port. In our example, the configuration is applied to Ethernet2/24, but the same approach can be used for any interface where you need to enforce QoS and PFC settings. The first command, priority-flow-control mode auto, enables Priority Flow Control (PFC) on the interface in auto-negotiation mode. This means the interface will automatically negotiate PFC with its link partner, allowing for lossless traffic handling by pausing specific traffic classes instead of dropping packets. The priority-flow-control watch-dog command enables the PFC watchdog for this interface, which ensures that if any PFC pause frames are stuck or persist for too long, the watchdog will take corrective action to prevent a deadlock situation. This helps maintain the overall health of the network by preventing traffic congestion or blockages due to PFC-related issues. Lastly, the service-policy type qos input QOS\_CLASSIFICATION command applies the QoS classification policy on incoming traffic, ensuring that packets are classified and marked according to their DSCP values as defined in the QOS\_CLASSIFICATION policy. This classification enables downstream QoS treatment, including proper queuing, scheduling, and priority handling. 

```
interface Ethernet 2/24
  priority-flow-control mode auto
  priority-flow-control watch-dog
  service-policy type qos input QOS\_CLASSIFICATION
```

**Example 11-7:** _Interface Level Configuration._

**References:**
* https://nwktimes.blogspot.com/2025/04/congestion-avoidance-ai-fabric-part-i.html
* https://nwktimes.blogspot.com/2025/04/congestion-avoidance-in-ai-fabric-part.html
* https://nwktimes.blogspot.com/2025/04/congestion-avoidance-in-ai-fabric-part_15.html
