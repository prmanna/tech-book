---
title: "Linux traceroute tool"
bookCollapseSection: true
weight: 10
---

# Intro
The linux traceroute tool is been great use for network engineer for their troubleshooting. The first version of this tool has been introduced 25 years ago. It's relying on the TTL field of the ip-header, keep on incrementing starting on TTL as 1. Along the way, each router decreases the TTL by one, and when it hits zero, the router sends back an ICMP 'time exceeded' message, revealing its identity.
The modern network has evloved over time. Now a ways, it has always the multi-path support for destinations, NAT, etc. Hence, traceroute needs to enhanced to accomodate these use-cases.

There are two implementations emerged. lets go little deeper for both the implementations.
1. paris-traceroute (https://paris-traceroute.net/about/)
2. dublin-traceroute (https://dublin-traceroute.net/)

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/flow-based-load-balancing.png)

## Paris Traceroute

## Dublin Traceroute


**References:**
* [Paris Traceroute](https://paris-traceroute.net/about/) 
* [Dublin Traceroute](https://dublin-traceroute.net/)
