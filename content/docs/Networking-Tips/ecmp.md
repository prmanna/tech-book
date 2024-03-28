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

## How Do I Solve the Hash Polarization Problem?
Hash polarization, also known as hash imbalance, indicates that traffic is unevenly load balanced after being hashed twice or more. This situation is common when hash operations are performed across devices multiple times. For example, a device performs ECMP hashing and forwards traffic to two or more connected devices, which then perform ECMP or Eth-Trunk hashing again. Hash polarization may also occur if the outbound interfaces of ECMP routes are multiple Eth-Trunk interfaces on the same device.
The implementation of the hash function on switches heavily depends on chips. Therefore, hash polarization may occur if switches using the same type of chips are located at adjacent network layers. If both Eth-Trunk hashing and ECMP hashing exist on the same device, hash polarization may also occur.
Therefore, if ECMP or Eth-Trunk hashing is deployed on a multi-layer network, consider the risk of hash polarization.
### Suggestions
If load imbalance or hash polarization occurs during traffic forwarding, you can adjust the hash algorithm on the device to resolve the problem. 
* Hash algorithm: is configured by specifying the hash-mode hash-mode-id parameter.
* Seed value: is configured by specifying the seed seed-data parameter. If devices from multiple vendors exist on the network, you are advised to configure the same seed value for these devices.
* Offset: is configured by specifying the universal-id universal-id parameter. Typically, one hash algorithm corresponds to one offset. When devices from multiple vendors exist on the network, you are advised to configure the same offset for these devices.

## Resilient Hashing
When a next hop fails or is removed from an ECMP pool, the hashing or hash bucket assignment can change. For deployments where there is a need for flows to always use the same next hop, like TCP anycast deployments, this can create session failures.

Resilient hashing is an alternate mechanism for managing ECMP groups. The ECMP hash performed with resilient hashing is exactly the same as the default hashing mode. Only the method in which next hops are assigned to hash buckets differs — they’re assigned to buckets by hashing their header fields and using the resulting hash to index into the table of 2^n hash buckets. Since all packets in a given flow have the same header hash value, they all use the same flow bucket.

**References:**
* https://docs.nvidia.com/networking-ethernet-software/cumulus-linux-43/Layer-3/Routing/Equal-Cost-Multipath-Load-Sharing-Hardware-ECMP/
* https://support.huawei.com/enterprise/en/doc/EDOC1100086965
