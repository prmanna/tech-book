---
title: "TCP Congestion Control"
bookCollapseSection: true
weight: 10
---

## Intro
TCP is a protocol that is used to transmit information from one computer on the internet to another, and is the protocol I’ll be focused on in this post. What distinguishes TCP from other networking protocols is that it guarantees 100% transmission. This means that if you send 100kb of data from one computer to another using TCP, all 100kb will make it to the other side.
This property of TCP is very powerful and is the reason that many network applications we use, such as the web and email are built on top of it.
The way TCP is able to accomplish this goal of trasmitting all the information that is sent over the wire is that for every segment of data that is sent from party A to party B, party B sends an “acknowledgement” segment back to party A indicating that it got that message.

## When does congestion happen?
Congestion is problem in computer networks because at the end of the day, information transfer rates are limited by physical channels like ethernet cables or cellular links, and on the internet, many individual devices are connected to these links.

## Detour: What is a link?
Before I dive into what some solutions to this problem are, I want to be a little bit more specific about the properties of links. There are three important details to know about a link:
* delay (milliseconds) - the time it takes for one packet to get from the beginning to the end of a link
* bandwidth (megabits/second) - the number of packets that can get through the link in a second
* queue - the size of the queue for storing packets waiting to be sent out if a link is full, and the strategy for managing that queue when it hits its capacity
Using the analogy of the link as a pipe, you can think of the delay as the length of the pipe, and the bandwidth as the circumference of the pipe. An important statistic about a link is the bandwidth-delay product (BDP). 

If senders are sending more bytes than the BDP, the link’s queue will fill and eventually start dropping packets.

## Approaches
There are two main indicators: packet loss and increased round trip times for packets. When congestion happens, queues on links begin to fill up, and packets get dropped. If a sender notices packet loss, it’s a pretty good indicator that congestion is occuring. Another consequence of queues filling up though is that if packets are spending more time in a queue before making it onto the link, the round trip time, which measures the time from when the sender sends a segment out to the time that it receives an acknowledgement, will increase.
While today there are congestion control schemes that take into account both of these indicators, in the original implementations of congestion control, only packet loss was used.

Therefore, in addition to being able to avoid congestion, congestion control approaches need to be able to “explore” the available bandwidth.

## The Congestion Window
A key concept to understand about any congestion control algorithm is the concept of a congestion window. The congestion window refers to the number of segments that a sender can send without having seen an acknowledgment yet. If the congestion window on a sender is set to 2, that means that after the sender sends 2 segments, it must wait to get an acknowledgment from the receiver in order to send any more. The congestion window is often referred to as the “flight size”, because it also corresponds to the number of segments “in flight” at any given point in time.

The higher the congestion window, more packets you’ll be able to get across to the receiver in the same time period. To understand this intuitively, if the delay on the network is 88ms, and the congestion window is 10 segments, you’ll be able to send 10 segments for every round trip (88*2 = 176 ms), and if it’s 20 segments, you’ll be able to send 20 segments in the same time period.

Of course, the risk with raising the congestion window too high is that it will lead to congestion. The goal of a congestion control algorithm, then, is to figure out the right size congestion window to use.

From a theoretical perspective, the right size congestion window to use is the bandwidth-delay product of the link, which as we discussed earlier is the full capacity of the link. The idea here is that if the congestion window is equal to the BDP of the link, it will be fully utilized, and not cause congestion.

