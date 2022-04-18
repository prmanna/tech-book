---
title: "SystemDesign-Tips"
bookCollapseSection: true
---

# System-Tips
---
In this sections, all the essential concents will be described.

### Storage
1. Disk - HDD(Hard-disk drive) and SSD(solid state drive). SSD is faster than HDD, hence costlier also. Persistent Storage.
2. Memory - RAM (Random access momory). Volatile storage

### Latency and Throughput
1. Latency - Time it takes for a certain operation to complete, unit msec or sec.
* Reading 1 MB from RAM: 250 us (0.25ms)
* Reading 1 MB from SSD: 1,000 ps (1 ms)
* Reading 1 MB from HDD: 20,000 is (20 ms)
* Transfer 1 MB over Network: 10,000 pus (10 ms)
* Inter-Continental Round Trip: 150,000 ps (150 ms)

2. Throughput - The number of operations that a system can handle properly per time unit. For instance the throughput of a sec measured in requests per second (RPS or QPS).

### Availability
