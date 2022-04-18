---
title: "SystemDesign-Tips"
bookCollapseSection: true
---

# System-Tips
---
In this sections, all the essential concents will be described.

### Storage
1. **Disk** - **HDD**(Hard-disk drive) and **SSD**(solid state drive). SSD is faster than HDD, hence costlier also. Persistent Storage.
2. **Memory** - RAM (Random access momory). Volatile storage

### Latency and Throughput
1. **Latency** - Time it takes for a certain operation to complete, unit msec or sec.
* Reading 1 MB from RAM: 250 us (0.25ms)
* Reading 1 MB from SSD: 1,000 ps (1 ms)
* Reading 1 MB from HDD: 20,000 is (20 ms)
* Transfer 1 MB over Network: 10,000 pus (10 ms)
* Inter-Continental Round Trip: 150,000 ps (150 ms)

2. **Throughput** - The number of operations that a system can handle properly per time unit. For instance the throughput of a sec measured in requests per second (RPS or QPS).

### Availability
1. **Availability** - The odds of a particular server or service being up and running at any point in time, usually measured in percentages. A server that has 99% availability will be operational 99% of the time (this would be described as having two nines of availability).
2. **High Availability** - Used to describe systems that have particularly high levels of availability, typically 5 nines or more; sometimes abbreviated "HA", 
3. **Nines** - Typically refers to percentages of uptime. 
   For example, 5 nines of availability means an uptime of 99.999% of the time. Below are the downtimes expected per year depending on those 9s: 
   
     - 99% (two 9s): 87.7 hours 
   
     - 99.9% (three 9s): 8.8 hours 
   
     - 99.99%: 52.6 minutes 
   
     - 99.999%: 5.3 minutes 
