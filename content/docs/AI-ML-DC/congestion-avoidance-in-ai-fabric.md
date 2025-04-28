---
title: "Congestion Avoidance in AI Fabric"
bookCollapseSection: true
weight: 10
---

## Intro
### Congestion Avoidance in AI Fabric - Part I: Explicit Congestion Notification (ECN)

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

[![](https://blogger.googleusercontent.com/img/a/AVvXsEgEGfIJg1fopCmHJHE-fU1Gq6JY0x6cS5Ena4NKHOvsLSqy-ATvdlL24btFkroyBpWhtN3YkcSo5w39Wopf_CfW81BePQ-yyGtsqKuOD71d3sed4MPVErnikUbHeXcfx3CvtSH2SoWSktUubCiKwF0hY4mHRjAbluW7xC3xUbt__EmBAupgD5zWFXkkFFU=w640-h354)](https://blogger.googleusercontent.com/img/a/AVvXsEgEGfIJg1fopCmHJHE-fU1Gq6JY0x6cS5Ena4NKHOvsLSqy-ATvdlL24btFkroyBpWhtN3YkcSo5w39Wopf_CfW81BePQ-yyGtsqKuOD71d3sed4MPVErnikUbHeXcfx3CvtSH2SoWSktUubCiKwF0hY4mHRjAbluW7xC3xUbt__EmBAupgD5zWFXkkFFU)

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

[![](https://blogger.googleusercontent.com/img/a/AVvXsEjGFDfe6_ew7DFA32PL5Mb8ovaQW5FN_NTzGwuT0N4HdiBIvd4iujPwD11SGvd1Mg-jeRaHoLjlJ4kTRanKWCAFLiODCSkmN4Dm8kRYMi-LTIgpswIFfecH6k5nfxZ5y8oUJXfyFoRhVkhQUbG9BghDR4OgPmigSX_n3tDT9RYPBzV6ZALm2GnXyjA-2tc=w640-h354)](https://blogger.googleusercontent.com/img/a/AVvXsEjGFDfe6_ew7DFA32PL5Mb8ovaQW5FN_NTzGwuT0N4HdiBIvd4iujPwD11SGvd1Mg-jeRaHoLjlJ4kTRanKWCAFLiODCSkmN4Dm8kRYMi-LTIgpswIFfecH6k5nfxZ5y8oUJXfyFoRhVkhQUbG9BghDR4OgPmigSX_n3tDT9RYPBzV6ZALm2GnXyjA-2tc)

**Figure 11-2:** _Congested Egress Interface – ECN Congestion Experienced._

After receiving a data packet marked with ECN 11, the destination RDMA NIC must inform the sender about network congestion. This is accomplished using a Congestion Notification Packet (CNP), which provides feedback at the RDMA transport layer. 

1. Upon detecting the ECN  11 in the IP header’s ToS bits of an incoming packet, the RDMA NIC on Host-2 generates a CNP message. It sets the OpCode in the IBTH header to 0x81, identifying the message as a CNP. Besides the FECN bit is set to 1, indicating that the NIC experienced congestion. The Queue Pair number used for the original memory copy (e.g., 0x123456) is reused, ensuring the feedback reaches the correct sender-side transport context. In the IP header, the DSCP field is set to 48, allowing switches to distinguish the CNP from standard RoCEv2 data traffic.
2. When the CNP reaches the Rail switch (interface Ethernet2/24), it is classified based on DSCP 48, which is associated with CNP traffic in the QoS configuration.
3. DSCP 48 maps the packet to QoS group 7, which is reserved for congestion feedback signaling.
4. QoS group 7 is associated with strict-priority egress queue 7, ensuring that CNP packets are forwarded with the highest priority. This guarantees that congestion signals are not delayed behind other types of traffic.
5. The switch forwards the CNP to the originating NIC on Host-1. Because of the strict-priority handling, the feedback arrives quickly even during severe congestion.
6. Upon receiving the CNP, the sender-side RDMA NIC reduces its transmission rate for the affected Queue Pair by increasing inter-packet delay. This is achieved by holding outgoing packets longer in local buffers, effectively reducing traffic injection into the congested fabric.

As transmission rates decrease, the pressure on the egress queue at the Rail switch’s interface Ethernet1/14 (connected to Host-2) is gradually relieved. Buffer occupancy falls below the WRED Minimum Threshold, ending ECN marking. Once congestion is fully cleared, the RDMA NIC slowly ramp up its transmission rate. This gradual marking strategy helps prevent sudden traffic loss and gives the sender time to react by adjusting its sending rate before the buffer overflows.

[![](https://blogger.googleusercontent.com/img/a/AVvXsEiSvmMWb7AsysqD6cV8J4poDQBtML_OhbUIOhZjZBFhIfLuj0Zbq3z7wZ1k61an0mnMCmX4QK1gmJEhllLJKvkFVG9XEUksXne26HZH7DAqNbQ8_ZbBLvVUZN76GGCBXsfgOsoIE2lqTv6W7_sfDXJStcN1JMbN9MPTSlHzM1F-VuGT_NFKZbuhDf0ebeg=w640-h258)](https://blogger.googleusercontent.com/img/a/AVvXsEiSvmMWb7AsysqD6cV8J4poDQBtML_OhbUIOhZjZBFhIfLuj0Zbq3z7wZ1k61an0mnMCmX4QK1gmJEhllLJKvkFVG9XEUksXne26HZH7DAqNbQ8_ZbBLvVUZN76GGCBXsfgOsoIE2lqTv6W7_sfDXJStcN1JMbN9MPTSlHzM1F-VuGT_NFKZbuhDf0ebeg)

**Figure 11-3:** _Receiving NIC Generates CNP - Sender NIC Delay Transmit._  
  
Next post describes DSCP-Based Priority Flow Control (PFC) use cases and operation.

### Congestion Avoidance in AI Fabric - Part II: Priority Flow Control (PFC)

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

  

• xOFF threshold: Triggers a pause when buffer usage exceeds this level. 

• xON threshold: Triggers a resume when the buffer has drained sufficiently.

  

Once priority queue 3 on ingress interface E3/24 exceeds its xOFF threshold, the switch takes immediate action to prevent packet loss by generating a PFC pause message targeted at the sender. The sender in this case is Spine Switch 1, which is sending traffic to Rail Switch C, over interface E3/24, for delivery to GPU-3.

  

#### Step 3: Generating a PFC Pause Frame (MAC Control Frame)

  

To pause the sender, Rail Switch C generates an Ethernet MAC Control frame with:

  

• Ethertype  0x8808: This indicates a MAC Control frame, used for pause-related operations (standardized in IEEE 802.3x). Inside this frame, a PFC opcode (0x0101) specifies it's a Priority-based Pause (PFC) message.

  

• Class Enable Vector (CEV): This 8-bit field indicates which priority queues should be paused. Each bit corresponds to one of the 8 possible traffic classes (0–7). For example, if bit 3  is set to 1, it tells the sender to pause traffic for priority queue 3 only, while all other bits remain 0. In our case, the CEV is 0x1000. Note that the right-most bit represents queue 0.

  

• Quanta Field(s): For each enabled priority (bit set to 1), a corresponding quanta value is specified. This value defines the duration of the pause, measured in units of 512 bit times.

  

For a 400 Gbps interface:

  

• 1 bit time = 1 / 400,000,000,000 seconds ≈ 2.5 picoseconds

• 1 quanta = 512 × 2.5 ps = 1.28 nanoseconds

• If the pause quanta is set to maximum value 0xFFFF (65535), the pause duration is roughly 83.9 microseconds.

  

This pause frame is sent back to the sender Spine Switch 1. Since the DSCP-based classification maps back to priority queue 3, and the switches share the same mapping, Spine Switch 1 will interpret this correctly.

  

#### Step 4: Spine Switch 1 Pauses Transmission on Priority Queue 3

  

Upon receiving the PFC frame on its ingress interface E3/24 connected to Rail Switch C, Spine Switch 1 examines the class enable vector.

  

• Since bit 3 is set, the switch knows to pause transmission of all frames mapped to priority queue 3 (DSCP value 24 in our example) on egress interface E3/24.

  

Traffic for other priority queues continues unaffected. 

  

Spine Switch 1 holds off transmission of priority 3 traffic until it receives a subsequent PFC frame with quanta = 0, indicating “resume,” or a pause duration timeout occurs, after which the switch resumes sending unless another pause is received.

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhAVViRoNi3YLc0xGnKXVG-YrfFaoTdcyZoGn424shMUS36ueU-PEw-ZplAzu2bZsuNIeslQaV2UNMfwMVd5L7hrIwwcTjWPSaMJ24jgAMhr71vv5_oZ_WbKxESr1v-f2047zefKGul0bAXc8qd9EDkFT3092Nsg0xMM1SrAMuVG2IQ5nTNjgnGF7OqnC4)](https://blogger.googleusercontent.com/img/a/AVvXsEhAVViRoNi3YLc0xGnKXVG-YrfFaoTdcyZoGn424shMUS36ueU-PEw-ZplAzu2bZsuNIeslQaV2UNMfwMVd5L7hrIwwcTjWPSaMJ24jgAMhr71vv5_oZ_WbKxESr1v-f2047zefKGul0bAXc8qd9EDkFT3092Nsg0xMM1SrAMuVG2IQ5nTNjgnGF7OqnC4)

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

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEgPep4YyHUju7sIns_r6uHabKnz6pXAPJHUt4su-N26Chxhtgd16aRt6HXtfktI0rxfAhmVkeVVSilVqz8pBDEV_gM1DRvoqgv7WGH-7L_NySvhuafZ6PL5RKyxnxYF-R3DNb64F9SFmYj7J2i-kfU0VNPEpnldnmC40b2pU-0QLAoGri3HjJWj_ZutKzA=w640-h350)](https://blogger.googleusercontent.com/img/a/AVvXsEgPep4YyHUju7sIns_r6uHabKnz6pXAPJHUt4su-N26Chxhtgd16aRt6HXtfktI0rxfAhmVkeVVSilVqz8pBDEV_gM1DRvoqgv7WGH-7L_NySvhuafZ6PL5RKyxnxYF-R3DNb64F9SFmYj7J2i-kfU0VNPEpnldnmC40b2pU-0QLAoGri3HjJWj_ZutKzA)

  

Figure 11-5: Priority Flow Control – Cascading Effect.

  

  

#### Steps 9a – 14: Downstream Resume and Congestion Recovery

  

Figure 11-6 illustrates how the PFC-based congestion recovery process extends from Rail Switches A and B all the way to the GPU NICs, while simultaneously resolving the initial congestion at Rail Switch C.

  

As a result of the earlier PFC pause frames:

  

• Rail Switch A and Rail Switch B have paused sending priority queue 3 traffic to Spine Switch 1.

• In turn, Spine Switch 1 has paused its own egress traffic toward Rail Switch C on interface E3/24.

  

This pause allows queue 3 on Rail Switch C’s egress interface E12/24 (toward GPU-3) to drain, as no new traffic is arriving, and the GPU continues to consume incoming data.

  

Once the buffer utilization for priority queue 3 drops below the configured xON threshold, Rail Switch C initiates congestion recovery.

  

• It sends a MAC Control Frame (Ethertype 0x8808) back to Spine Switch 1.

• The class enable vector has bit 3 set (indicating priority queue 3).

• The quanta value is set to 0, signaling that it is now safe to resume transmission.

  

Upon receiving this resume message, Spine Switch 1 can begin sending traffic again on priority queue 3, restoring throughput toward GPU-3 and continuing the flow of RDMA traffic through the network. This recovery mechanism operates consistently across the entire AI fabric.

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEiXiAPakEdpNy7ndyVtX2qSJsVLjJUE-zdVdtl5IQCkkI0LscBp5gI1IEbhpvntA7e1AL2pobhkItx1N3-AHYQApU-JrP2mf0N9aJjhmiFTgfcSt0Qj3w3NAQuw2iURkHaG2PUnUpSxpsCLyZC1m2L-I6upBVi0m_mFkeASQxLeaX-P9nvJsZfvpWvM8r8)](https://blogger.googleusercontent.com/img/a/AVvXsEiXiAPakEdpNy7ndyVtX2qSJsVLjJUE-zdVdtl5IQCkkI0LscBp5gI1IEbhpvntA7e1AL2pobhkItx1N3-AHYQApU-JrP2mf0N9aJjhmiFTgfcSt0Qj3w3NAQuw2iURkHaG2PUnUpSxpsCLyZC1m2L-I6upBVi0m_mFkeASQxLeaX-P9nvJsZfvpWvM8r8)

  

Figure 11-6: Priority Flow Control – PCIe Bus Congested: Cascading Effect.

  

  

### LLDP with DCBX

  

PFC negotiation is performed using the Link Layer Discovery Protocol (LLDP), which carries Data Center Bridging eXchange (DCBX) Type-Length-Value (TLV) structures. At the time of writing, DCBX exists in two versions: IEEE and CEE. The IEEE mode (defined in 802.1Qbb and 802.1Qaz) is standards-based and supported by most modern data center switches from various vendors. This mode is also known as DCBXv2. Some older Cisco Nexus models support only the Cisco/Converged Enhanced Ethernet (CEE) mode. Capture 11-1 shows the packet format of a standards-based IEEE DCBX TLV within an LLDP message.

  

[![](https://blogger.googleusercontent.com/img/a/AVvXsEgjNgAZ_hpdDNhHqRpGEIqd6qSthS6djkRbHkfzoKI7iyRNqbgyStFo8jLIjdO3caJuuYvbsWYlQTUvRsasvVgUd5EzXa46LzDnDJivcL1OEfzWiNMp2Rrc7f7_edk9y-5x9J7r0VnM8DbvBydW_arRaYo_e-FZSdmmpHpcHYnPkyPkKdzEOXrYlqkz9rA=w640-h368)](https://blogger.googleusercontent.com/img/a/AVvXsEgjNgAZ_hpdDNhHqRpGEIqd6qSthS6djkRbHkfzoKI7iyRNqbgyStFo8jLIjdO3caJuuYvbsWYlQTUvRsasvVgUd5EzXa46LzDnDJivcL1OEfzWiNMp2Rrc7f7_edk9y-5x9J7r0VnM8DbvBydW_arRaYo_e-FZSdmmpHpcHYnPkyPkKdzEOXrYlqkz9rA)

Capture 11-1: PCF: LLDP with IEEE DBCXv2 TLV .

**References:**
* https://nwktimes.blogspot.com/2025/04/congestion-avoidance-ai-fabric-part-i.html
* https://nwktimes.blogspot.com/2025/04/congestion-avoidance-in-ai-fabric-part.html
* https://nwktimes.blogspot.com/2025/04/congestion-avoidance-in-ai-fabric-part_15.html
