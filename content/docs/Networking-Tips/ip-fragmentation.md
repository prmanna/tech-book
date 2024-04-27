---
title: "IP Fragmentation - IPv4 & IPv6"
bookCollapseSection: true
weight: 10
---

# Intro
Like IPv4, IPv6 fragmentation divides an IPv6 packet into smaller packets to facilitate transmission across networks with a smaller Maximum Transmission Unit (MTU). Unlike IPv4, fragmentation is not mandatory in IPv6, as all networks support an MTU of at least 1280 bytes.

* Unlike IPv4, IPv6 relies on the source device instead of intermediary routers for fragmentation
* Unlike IPv4, an IPv6 router does not fragment a packet unless it is the packet’s source. Intermediate nodes (routers) do not fragment. You will see how an IPv6 device fragments packets when it is the source of the packet with the use of extension headers.
An IPv6 router drops packets too large for the egress interface and sends an ICMPv6 Packet Too Big message back to the source. Packet Too Big messages include the link’s MTU size in bytes so the source can resize the packet. Therefore, using the largest packet size supported by all the links from the source to the destination is preferable. Path MTUs (PMTUs) are used for this purpose. 

## Path MTU Discovery:
In addition, IPv6 nodes can use the Path MTU Discovery (PMTUD) mechanism to dynamically determine the maximum MTU size along the path to a destination. PMTUD sends packets with the “Don’t Fragment” (DF) flag set and progressively reduces the packet size until a smaller MTU is found. Once the maximum MTU size is determined, the source node can adjust its packet size accordingly to avoid fragmentation.

While IPv6 fragmentation is a valuable mechanism for ensuring packet delivery over networks with smaller MTU sizes, it should be used sparingly. Minimizing the need for fragmentation through proper MTU configuration and utilizing PMTUD can help improve network performance and reliability.
![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ip-fragment/rsz_1ipv6_path_mtu_discovery.png)

## ICMP and ICMPv6
The Internet Control Messaging Protocol ( ICMP ) was initially introduced to aid network troubleshooting by providing tools to verify end-to-end reachability. ICMP also reports back errors on hosts. Unfortunately, due to its nature and lack of built-in security, it quickly became a target for many attacks. For example, an attacker can use ICMP REQUESTS for network reconnaissance.

ICMP’s lack of inherent security opened it up to some vulnerabilities. This results in many security teams blocking all ICMP message types, which harms useful ICMP features such as Path MTU. ICMP for v4 and v6 are entirely different. Unlike ICMP for IPv4, ICMPv6 is an integral part of v6 communication, and ICMPv6 has features required for IPv6 operation. For this reason, it is not possible to block ICMPv6 and all its message types. ICMPv6 is a legitimate part of V6; you must select what you can filter. ICMPv6 should not be completely filtered.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ip-fragment/rsz_1ip_fragmentation.png)

## Fragmentation is normal
Fragmentation is a normal process on packet-switched networks. It occurs when a large packet is received, and the corresponding outbound interface’s maximum transmission unit (MTU) size is too small. Fragmentation dissects the IP packet into smaller packets before transmission. The receiving host performs fragment reassembly and passes the complete IP packet up the protocol stack.

Fragmentation is an IP process; TCP and other layers above IP are not involved. Reassembly is intended in the receiving host, but in practice, it may be done by an intermediate router. For example, network address translation (NAT) may need to reassemble fragments to translate data streams. This is where some differences between fragmentation in IPv4 and IPv6 are apparent.

So, in summary. IP fragmentation occurs at the Internet Protocol (IP) layer. Packets are fragmented to pass through a link with a smaller maximum transmission unit (MTU) than the original packet size. The receiving host then reassembles fragments.


## Impact on networks
In networks with multiple parallel paths, technologies such as LAG and CEF split traffic according to a hash algorithm. All packets from the same flow are sent out on the same path to minimize packet reordering. IP fragmentation can cause excessive retransmissions when fragments encounter packet loss. This is because reliable protocols such as TCP must retransmit all fragments to recover from the loss of a single fragment.

## Fragmentation in IPv4 and IPv6
Fragmentation in IPv6 is splitting a single inbound IP datagram into two or more outbound IP datagrams. The IP header is copied from the original IP datagram into the fragments. With IPv6, special bits are set in the fragments’ IPv6 headers to indicate that they are not complete IP packets. In the case of IPv6, we use **IPv6 extension headers**. Then, the payload is spread across the fragments.
In IPv4, fragmentation is done **whenever required, at the destination or routers**, whereas, in IPv6, only the source, not the routers, is supposed to do fragmentation. This can only be done when the source knows the Maximum Transmission Unit (MTU) path. **The IPv6 “do not fragment” bit is always 1**, whereas the case is not the same in IPv4, and the ‘More fragment’ bit is the only flag in the fragmentation header, which is one bit. Two bits are reserved for future use, as shown in the picture below. The following diagram displays the Internet Protocol Version 6 Fragmentation Header.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ip-fragment/rsz_1ipv6_fragmentsiton_headre.png)
## IPv6 fragmentation example
In an IPv6 world, the IPv6 header length is limited to 40 bytes, yet the IPv4 header has a max of 60. The primary IPv6 header remains a fixed size. IPv6 has the concept of **extension headers** to add optional IP layer information. Special handling with IPv4 was controlled by “I**P options**,” but there are no IP options in IPv6. All options are moved to different types of extension headers.
The following IPv6 extension headers are currently defined.

* Routing – Extended routing, like IPv4 loose source route
* Fragmentation – Fragmentation and reassembly
* Authentication – Integrity and authentication, security
* Encapsulation – Confidentiality
* Hop-by-Hop Option – Special options that require hop-by-hop processing
* Destination Options – Optional information to be examined by the destination node

## The IPv6 fragment header
With IPv4, all this was contained in the IPv4 header. There is no separate fragment header in IPv4, and the fragment fields in the IPv4 header are moved to the **IPv6 fragment header.** The _“Don’t fragment_” bit (DF) is removed, and intermediate routers are not allowed to fragment. They only permitted end stations to create and reassemble fragments ([**RFC 2460)**, **not intermediate routers**](https://www.rfc-editor.org/rfc/rfc2460). The design decision stems from the performance hit that fragmentation imposes on nodes.

Routers are no longer required to perform packet fragmentation and reassembly, making dropped packets more significant than the router’s interface MTU. Instead, I**Pv6 hosts perform PMTU** to determine the maximum packet size for the entire path. When a packet hits an interface with a smaller MTU, the routers send back an **ICMPv6 type 2 error, known as Packet Too Big**, to the sending host. The sending host receives the error message, reduces the size of the sending packet, and tries again.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/ip-fragment/rsz_1ipv6_frme_examke.png)

**References:**
* [IPv6 Fragmentation](https://network-insight.net/2015/10/16/ipv6-fragmentation/) 
