---
title: "QoS"
bookCollapseSection: true
weight: 10
---

# Intro

## Traffic Shaping

The traffic shapers that we will look at here are commonly called token bucket shapers and are based on a token bucket algorithm which we will see can serve multiple purposes in achieving network QoS. We should note that frequently in networking literature such algorithms were also referred to as leaky bucket algorithms. In all but one case, that we will not be discussing here, these algorithms produce the same results. Use the token bucket and leaky bucket Wikipedia entries as a guide if you are reading older networking literature.

The basic token bucket traffic shaper is characterized by two parameters: a rate, r, and a bucket size, b. The rate can be in either bits or bytes per second and the bucket size can be in either bits or bytes.

The key pieces of the shaper are:

Token generator with rate r. This requires some type of relatively accurate timing mechanism. The granularity of this mechanism heavily influences the accuracy to which r can be approximated.

Token bucket with size b. This can be implemented with up/down counters or adders/subtractors in hardware or software.

A buffer (queue) to hold packets

Logic to control the release of packets based on packet size and current bucket fill.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/BasicShaper.png)

**References:**
