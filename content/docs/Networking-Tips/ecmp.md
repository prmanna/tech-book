---
title: "ECMP Load Balancing"
bookCollapseSection: true
weight: 10
---

# Intro

## ECMP Hashing
ECMP is classified into **per-flow load balancing** and **per-packet load balancing**.

**Per-flow** load balancing can ensure the packet sequence and ensure that the same data flow is forwarded according to the routing entry with the same next hop and different data flows are forwarded according to routing entries with different next hops.

When multiple routes are installed in the routing table, a hash is used to determine which path a packet follows.

Hashes on the following fields:
* IP protocol
* Ingress interface
* Source IPv4 or IPv6 address
* Destination IPv4 or IPv6 address

Further, hashes on these additional fields:
* Source MAC address
* Destination MAC address
* Ethertype
* VLAN ID

For TCP/UDP frames, also hashes on:
* Source port
* Destination port

**Per-packet** load balancing improves ECMP bandwidth utilization and evenly load balances traffic among equal-cost routes, but it cannot prevent out-of-order packet delivery. To resolve this issue, the device or terminal that receives traffic must support reassembly of out-of-order packets.

The following per-packet load balancing modes are supported:
**Random mode:** A route is randomly selected among multiple equal-cost routes to forward packets. When the IP address and MAC address of known unicast packets remain unchanged, configure random per-packet load balancing.
**Round-robin mode**: Each equal-cost route is used to forward packets in turn. When known unicast packets have a similar length, configure round-robin per-packet load balancing.

## Eth-Trunk/Bundle Load Balancing

Ethernet link aggregation, also known as Eth-Trunk, bundles multiple physical links into a logical link to increase link bandwidth. The bundled links back up each other, increasing reliability.

Eth-Trunk load balancing is classified into per-packet load balancing and per-flow load balancing.
* Per-packet load balancing improves Eth-Trunk bandwidth utilization, but it cannot prevent out-of-order packet delivery. To resolve this issue, the device or terminal that receives traffic must support reassembly of out-of-order packets. The following per-packet load balancing modes are supported:
  + Random mode: The outbound interface of packets is selected randomly based on the time when packets reach the Eth-Trunk. When the IP address and MAC address of known unicast packets remain unchanged, configure random per-packet load balancing.
  + Round-robin mode: Each member interface of an Eth-Trunk forwards traffic in turn. When known unicast packets have a similar length, configure round-robin per-packet load balancing.
* Per-flow load balancing ensures the packet sequence and ensures that the same data flow is forwarded through the same physical link and different data flows are forwarded through different physical links. Table 1-2 describes the per-flow load balancing modes for different types of packets.

**References:**
