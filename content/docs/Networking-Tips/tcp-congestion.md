---
title: "TCP Congestion Control"
bookCollapseSection: true
weight: 10
---

# Intro
TCP is a protocol that is used to transmit information from one computer on the internet to another, and is the protocol I’ll be focused on in this post. What distinguishes TCP from other networking protocols is that it guarantees 100% transmission. This means that if you send 100kb of data from one computer to another using TCP, all 100kb will make it to the other side.
This property of TCP is very powerful and is the reason that many network applications we use, such as the web and email are built on top of it.
The way TCP is able to accomplish this goal of trasmitting all the information that is sent over the wire is that for every segment of data that is sent from party A to party B, party B sends an “acknowledgement” segment back to party A indicating that it got that message.

# When does congestion happen?
Congestion is problem in computer networks because at the end of the day, information transfer rates are limited by physical channels like ethernet cables or cellular links, and on the internet, many individual devices are connected to these links.

# Detour: What is a link?
Before I dive into what some solutions to this problem are, I want to be a little bit more specific about the properties of links. There are three important details to know about a link:
* delay (milliseconds) - the time it takes for one packet to get from the beginning to the end of a link
* bandwidth (megabits/second) - the number of packets that can get through the link in a second
* queue - the size of the queue for storing packets waiting to be sent out if a link is full, and the strategy for managing that queue when it hits its capacity
Using the analogy of the link as a pipe, you can think of the delay as the length of the pipe, and the bandwidth as the circumference of the pipe. An important statistic about a link is the bandwidth-delay product (BDP). 

If senders are sending more bytes than the BDP, the link’s queue will fill and eventually start dropping packets.

# Approaches
There are two main indicators: packet loss and increased round trip times for packets. When congestion happens, queues on links begin to fill up, and packets get dropped. If a sender notices packet loss, it’s a pretty good indicator that congestion is occuring. Another consequence of queues filling up though is that if packets are spending more time in a queue before making it onto the link, the round trip time, which measures the time from when the sender sends a segment out to the time that it receives an acknowledgement, will increase.
While today there are congestion control schemes that take into account both of these indicators, in the original implementations of congestion control, only packet loss was used.

Therefore, in addition to being able to avoid congestion, congestion control approaches need to be able to “explore” the available bandwidth.

# Control-Based Algorithms

## The Congestion Window
A key concept to understand about any congestion control algorithm is the concept of a congestion window. The congestion window refers to the number of segments that a sender can send without having seen an acknowledgment yet. If the congestion window on a sender is set to 2, that means that after the sender sends 2 segments, it must wait to get an acknowledgment from the receiver in order to send any more. The congestion window is often referred to as the “flight size”, because it also corresponds to the number of segments “in flight” at any given point in time.
 
The higher the congestion window, more packets you’ll be able to get across to the receiver in the same time period. To understand this intuitively, if the delay on the network is 88ms, and the congestion window is 10 segments, you’ll be able to send 10 segments for every round trip (88*2 = 176 ms), and if it’s 20 segments, you’ll be able to send 20 segments in the same time period.

Of course, the risk with raising the congestion window too high is that it will lead to congestion. The goal of a congestion control algorithm, then, is to figure out the right size congestion window to use.

From a theoretical perspective, the right size congestion window to use is the bandwidth-delay product of the link, which as we discussed earlier is the full capacity of the link. The idea here is that if the congestion window is equal to the BDP of the link, it will be fully utilized, and not cause congestion.

## TCP Tahoe
TCP Tahoe is a congestion control scheme that was invented back in the 80s, when congestion was first becoming a problem on the internet. The algorithm itself is fairly simple, and grows the congestion window in two phases.

### Phase 1
Slow Start: The algorithm begins in a state called “slow start”. In Slow Start, the congestion window grows by 1 every time an acknowledgement is received. This effectively doubles the congestion window on every round trip. If the congestion window is 4, four packets will be in flight at once, and when each of those packets is acknowledged, the congestion window will increase by 1, resulting in a window of size 8. This process continues until the congestion window hits a value called the “Slow Start Threshold” ssthresh. This is a configurable number.

### Phase 2
Congestion Avoidance: Once the congestion window has hit the ssthresh, it moves from “slow start” into congestion avoidance mode. In congestion avoidance, the congestion window increases by 1 on every round trip. So if the congestion window is 4, the window will increase to 5 after all four of those packets in flight have been acknowledged. This increases the window much more slowly.

If Tahoe detects that a packet is lost, it will resend the packet, the slow start threshold is updated to be half the current congestion window, the congestion window is set back to 1, and the algorithm goes back to slow start.

### Detecting Packet Loss & Fast Retransmit
There are two ways that a TCP sender could detect that a packet is lost.

The sender “times out”. The sender puts a timeout on every packet that is sent out into the wild, and when that timeout is hit without that packet having been acknowledged, it resends the packet and sets the congestion window to 1.

The receiver sends back “duplicate acks”. In TCP, receivers only acknowledge packets that are sent in order. If a packet is sent out of order, it will send out an acknowledgement for the last packet it saw in order. So, if a receiver has received segments 1,2, and 3, and then receives segment #5, it will ack segment #3 again, because #5 came in out of order. In Tahoe, if a sender receives 3 duplicate acks, it considers a packet lost. This is considered “Fast Retransmit”, because it doesn’t wait for the timeout to happen.

There are a number of issues with this approach though, which is why it is no longer used today. In particular, it takes a really long time, especially on higher bandwidth networks, for the algorithm to actually take full advantage of the available bandwidth. This is because the window size grows pretty slowly after hitting the slow start threshold.
Another issue is that packet loss doesn’t necessarily mean that congestion is occuring–some links, like WiFi, are just inherently lossy. Reacting drastically by cutting the window size to 1 isn’t necessarily always appropriate.
A final issue is that this algorithm uses packet loss as the indicator for whether there’s congestion. If the packet loss is happening due to congestion, you are already too late–the window is too high, and you need to let the queues drain.

## TCP CUBIC
This algorithm was implemented in 2005, and is currently the default congestion control algorithm used on Linux systems. Like Tahoe, it relies on packet loss as the indicator of congestion. However, unlike Tahoe, it works far better on high bandwidth networks, since rather than increasing the window by 1 on every round trip, it uses, as the name would suggest, a cubic function to determine what the window size should be, and therefore grows much more quickly.

# Avoidance-Based Algorithms

## BBR(Bottleneck Bandwidth and RTT) - (Bufferbloat) 
This is a very recent algorithm developed by Google, and unlike Tahoe or CUBIC, uses delay as the indicator of congestion, rather than packet loss. The rough thinking behind this is that delays are a leading indicator of congestion–they occur before packets actually start getting lost. Slowing down the rate of sending before the packets get lost ends up leading to higher throughput.

# Active Queue Management

## Random Early Detection
Each router is programmed to monitor its own queue length and, when it detects that congestion is imminent, to notify the source to adjust its congestion window. RED, invented by Sally Floyd and Van Jacobson in the early 1990s.

RED is most commonly implemented such that it implicitly notifies the source of congestion by dropping one of its packets. The source is, therefore, effectively notified by the subsequent timeout or duplicate ACK. In case you haven’t already guessed, RED is designed to be used in conjunction with TCP, which currently detects congestion by means of timeouts (or some other means of detecting packet loss such as duplicate ACKs). As the “early” part of the RED acronym suggests, the gateway drops the packet earlier than it would have to, so as to notify the source that it should decrease its congestion window sooner than it would normally have. In other words, the router drops a few packets before it has exhausted its buffer space completely, so as to cause the source to slow down, with the hope that this will mean it does not have to drop lots of packets later on.

how RED decides when to drop a packet and what packet it decides to drop. To understand the basic idea, consider a simple FIFO queue. Rather than wait for the queue to become completely full and then be forced to drop each arriving packet (the tail drop policy), we could decide to drop each arriving packet with some drop probability whenever the queue length exceeds some drop level. This idea is called early random drop. The RED algorithm defines the details of how to monitor the queue length and when to drop a packet.

## Explicit Congestion Notification || IP & TCP Flags

While TCP’s congestion control mechanism was initially based on packet loss as the primary congestion signal, it has long been recognized that TCP could do a better job if routers were to send a more explicit congestion signal. That is, instead of dropping a packet and assuming TCP will eventually notice (e.g., due to the arrival of a duplicate ACK), any AQM algorithm can potentially do a better job if it instead marks the packet and continues to send it along its way to the destination. This idea was codified in changes to the IP and TCP headers known as Explicit Congestion Notification (ECN), as specified in RFC 3168.

Specifically, this feedback is implemented by treating **two bits in the IP TOS** field as ECN bits. One bit is set by the source to indicate that it is **ECN-capable**, that is, able to react to a congestion notification. This is called the ECT bit (ECN-Capable Transport). The other bit is set by routers along the end-to-end path when congestion is encountered, as computed by whatever AQM algorithm it is running. This is called the **CE bit (Congestion Encountered)**.

In addition to these two bits in the IP header (which are transport-agnostic), ECN also includes the addition of **two optional flags to the TCP header**. The first, **ECE (ECN-Echo/Experienced)**, communicates from the receiver to the sender that it has received a packet with the CE bit set. The second, **CWR (Congestion Window Reduced)** communicates from the sender to the receiver that it has reduced the congestion window.

# Beyond TCP
## Datacenters (DCTCP)
There have been several efforts to optimize TCP for cloud datacenters, where Data Center TCP was one of the first. There are several aspects of the datacenter environment that warrant an approach that differs from more traditional TCP. These include:

* Round trip time for intra-DC traffic are small;
* Buffers in datacenter switches are also typically small;
* All the switches are under common administrative control, and thus can be required to meet certain standards;
* A great deal of traffic has low latency requirements;
* That traffic competes with high bandwidth flows.

It should be noted that DCTCP is not just a version of TCP, but rather, a system design that changes both the switch behavior and the end host response to congestion information received from switches.

The central insight in DCTCP is that using loss as the main signal of congestion in the datacenter environment is insufficient. By the time a queue has built up enough to overflow, low latency traffic is already failing to meet its deadlines, negatively impacting performance. Thus DCTCP uses a version of ECN to provide an early signal of congestion. But whereas the original design of ECN treated an ECN marking much like a dropped packet, and cut the congestion window in half, DCTCP takes a more finely-tuned approach. DCTCP tries to estimate the fraction of bytes that are encountering congestion rather than making the simple binary decision that congestion is present. It then scales the congestion window based on this estimate. The standard TCP algorithm still kicks in should a packet actually be lost. The approach is designed to keep queues short by reacting early to congestion while not over-reacting to the point that they run empty and sacrifice throughput.

The key challenge in this approach is to estimate the fraction of bytes encountering congestion. Each switch is simple. If a packet arrives and the switch sees the queue length (K) is above some threshold; e.g.,
K > (RTT * C) / 7

where C is the link rate in packets per second, then the switch sets the CE bit in the IP header. The complexity of RED is not required.

The receiver then maintains a boolean variable for every flow, which we’ll denote DCTCP.CE, and sets it initially to false. When sending an ACK, the receiver sets the ECE (Echo Congestion Experienced) flag in the TCP header if and only if DCTCP.CE is true. It also implements the following state machine in response to every received packet:

If the CE bit is set and DCTCP.CE=False, set DCTCP.CE to True and send an immediate ACK.

If the CE bit is not set and DCTCP.CE=True, set DCTCP.CE to False and send an immediate ACK.

Otherwise, ignore the CE bit.

## HTTP Performance (QUIC)
HTTP has been around since the invention of the World Wide Web in the 1990s and from its inception it has run over TCP. HTTP/1.0, the original version, had quite a number of performance problems due to the way it used TCP, such as the fact that every request for an object required a new TCP connection to be set up and then closed after the reply was returned. HTTP/1.1 was proposed at an early stage to make better use of TCP. TCP continued to be the protocol used by HTTP for another twenty-plus years.

In fact, TCP continued to be problematic as a protocol to support the Web, especially because a reliable, ordered byte stream isn’t exactly the right model for Web traffic. In particular, since most web pages contain many objects, it makes sense to be able to request many objects in parallel, but TCP only provides a single byte stream. If one packet is lost, TCP waits for its retransmission and successful delivery before continuing, while HTTP would have been happy to receive other objects that were not affected by that single lost packet. Opening multiple TCP connections would appear to be a solution to this, but that has its own set of drawbacks including a lack of shared information about congestion across connections.

Other factors such as the rise of high-latency wireless networks, the availability of multiple networks for a single device (e.g., Wi-Fi and cellular), and the increasing use of encrypted, authenticated connections on the Web also contributed to the realization that the transport layer for HTTP would benefit from a new approach. The protocol that emerged to fill this need was QUIC.

QUIC originated at Google in 2012 and was subsequently developed as a proposed standard at the IETF. It has already seen a solid amount of deployment—it is in most Web browsers, many popular websites, and is even starting to be used for non-HTTP applications. Deployability was a key consideration for the designers of the protocol. There are a lot of moving parts to QUIC—its specification spans three RFCs of several hundred pages—but we focus here on its approach to congestion control, which embraces many of the ideas we have seen to date in this book.

Like TCP, QUIC builds congestion control into the transport, but it does so in a way that recognizes that there is no single perfect congestion control algorithm. Instead, there is an assumption that different senders may use different algorithms. The baseline algorithm in the QUIC specification is similar to TCP NewReno, but a sender can unilaterally choose a different algorithm to use, such as CUBIC. QUIC provides all the machinery to detect lost packets in support of various congestion control algorithms.

## Multipath Transport
While the early hosts connected to the Internet had only a single network interface, it is common these days to have interfaces to at least two different networks on a device. The most common example is a mobile phone with both cellular and WiFi interfaces. Another example is datacenters, which often allocate multiple network interfaces to servers to improve fault tolerance. Many applications use only one of the available networks at a time, but the potential exists to improve performance by using multiple interfaces simultaneously. This idea of multipath communication has been around for decades and led to a body of work at the IETF to standardize extensions to TCP to support end-to-end connections that leverage multiple paths between pairs of hosts. This is known as Multipath TCP (MPTCP).

A pair of hosts sending traffic over two or more paths simultaneously has implications for congestion control. For example, if both paths share a common bottleneck link, then a naive implementation of one TCP connection per path would acquire twice as much share of the bottleneck bandwidth as a standard TCP connection. The designers of MPTCP set out to address this potential unfairness while also realizing the benefits of multiple paths. The proposed congestion control approach could equally be applied to other transports such as QUIC. 

**References:**
* https://tcpcc.systemsapproach.org/aqm.html
* https://squidarth.com/rc/programming/networking/2018/07/18/intro-congestion
