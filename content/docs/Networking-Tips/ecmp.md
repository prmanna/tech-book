---
title: "ECMP Load Balancing"
bookCollapseSection: true
weight: 10
---

# Intro

## ECMP Hashing
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

For TCP/UDP frames, Cumulus Linux also hashes on:
* Source port
* Destination port


**References:**
