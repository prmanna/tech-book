---
title: "QoS"
bookCollapseSection: true
weight: 10
---

# Intro

## Traffic Shaping

The traffic shapers that we will look at here are commonly called token bucket shapers and are based on a token bucket algorithm which we will see can serve multiple purposes in achieving network QoS. We should note that frequently in networking literature such algorithms were also referred to as leaky bucket algorithms. In all but one case, that we will not be discussing here, these algorithms produce the same results. Use the token bucket and leaky bucket Wikipedia entries as a guide if you are reading older networking literature.

**The basic token bucket traffic shaper is characterized by two parameters: a rate, r, and a bucket size, b. The rate can be in either bits or bytes per second and the bucket size can be in either bits or bytes.**

The key pieces of the shaper are:

* Token generator with rate r. This requires some type of relatively accurate timing mechanism. The granularity of this mechanism heavily influences the accuracy to which r can be approximated.

* Token bucket with size b. This can be implemented with up/down counters or adders/subtractors in hardware or software.

* A buffer (queue) to hold packets

Logic to control the release of packets based on packet size and current bucket fill.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/BasicShaper.png)

## Policing
### A Single Rate Three Color Marker

A policer called a single rate three color marker (srTCM) is defined in RFC2697 an informational RFC. The single rate means we will use a single token generator with a specified rate parameter. By three color they mean that packets will be marked with by three levels of compliance (Green, Yellow, Red) from compliant to most out of compliance.

This particular policer is defined by three traffic parameters, an update algorithm, and marking criteria. The three traffic parameters are:

* Committed Information Rate (CIR) measured in bytes of IP packets per second, i.e., it includes the IP header, but not link specific headers. This is the token rate.

* Committed Burst Size (CBS) measured in bytes. This is the token bucket size associated with the rate.

Excess Burst Size (EBS) measured in bytes. This is a bucket filled from the overflow of the first bucket hence the expression "excesses burst size". This bucket allows us to save up tokens from periods of inactivity for later use. The CBS and EBS must be configured so that at least one of them is larger than 0.

There are two modes of operation for the srTCM one, called Color-Aware works with packets that have been previously marked by another policer in the network, the other mode called Color-Blind works with packets that have not been previously marked. The Color-Blind mode of operation is shown in below.

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/MarkerSingle.png)

### Two rate three color marker

Another policer called a two rate three color marker (trTCM) is defined in RFC2698, an informational RFC. The two rate means we will use a two token generators each with a specified rate parameter. Once again the three colors Green, Yellow, Red indicate the level of compliance which gets translated into packet drop precedence when the packets get marked. The parameters for this policer are: the Peak Information Rate (PIR), the Peak Burst Size (PBS), the Committed Information Rate (CIR), and the Committed Burst Size (CBS). All use units similar to that of the srTCM. The initialization, update, and marking behavior for the Color-Blind mode is shown below.


![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/MarkerDouble.png)

**References:**

**References:**
