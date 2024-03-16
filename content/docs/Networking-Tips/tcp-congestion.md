---
title: "TCP Congestion Control"
bookCollapseSection: true
weight: 10
---

## Intro
TCP is a protocol that is used to transmit information from one computer on the internet to another, and is the protocol I’ll be focused on in this post. What distinguishes TCP from other networking protocols is that it guarantees 100% transmission. This means that if you send 100kb of data from one computer to another using TCP, all 100kb will make it to the other side.

This property of TCP is very powerful and is the reason that many network applications we use, such as the web and email are built on top of it.

The way TCP is able to accomplish this goal of trasmitting all the information that is sent over the wire is that for every segment of data that is sent from party A to party B, party B sends an “acknowledgement” segment back to party A indicating that it got that message.

