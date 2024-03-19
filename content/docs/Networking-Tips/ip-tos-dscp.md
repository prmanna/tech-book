---
title: "IP Precedence And TOS | DSCP"
bookCollapseSection: true
weight: 10
---

# Intro
8 Bits of Type of Service in IP Header.

## IP Precedence 
* RFC791/RFC1349 Interpretation

    ** Bits **
    **7-5 IP Precedence**

    111	Network Control
    110	Internetwork Control
    101	Critic/ECP
    100	Flash Override
    011	Flash
    010	Immediate
    001	Priority
    000	Routine

    **Bits**
    4   (1 = Low Delay; 0 = Normal Delay)
    3   (1 = High Throughput; 0 = Normal Throughput)
    2   (1 = High Reliability; 0 = Normal Reliability)
    1   (1 = Minimise monetary cost (RFC 1349))
    0   (Must be 0)

## TOS | DSCP (Differentiated Services Code Point)
RFC 2474 (Differentiated Services) Interpretation
**Bits**
7-2	DSCP
1-0	ECN (Explicit Congestion Notification)

### Default Forwarding (DF) PHB
Typically best-effort traffic
The recommended DSCP for DF is 0

### Expedited Forwarding (EF) PHB 
Dedicated to low-loss, low-latency traffic
The recommended DSCP for EF is 101110B (46 or 2E(hex))

### Assured Forwarding (AF) PHB 
Gives assurance of delivery under prescribed conditions


| Drop  
probability | Class 1 | Class 2 | Class 3 | Class 4 |
| --- | --- | --- | --- | --- |
| Low | AF11 (DSCP 10) 001010 | AF21 (DSCP 18) 010010 | AF31 (DSCP 26) 011010 | AF41 (DSCP 34) 100010 |
| Medium | AF12 (DSCP 12) 001100 | AF22 (DSCP 20) 010100 | AF32 (DSCP 28) 011100 | AF42 (DSCP 36) 100100 |
| High | AF13 (DSCP 14) 001110 | AF23 (DSCP 22) 010110 | AF33 (DSCP 30) 011110 | AF43 (DSCP 38) 100110 |

### Class Selector PHBs
This maintains backward compatibility with the IP precedence field.

PHB == Per-Hop-Behavior

