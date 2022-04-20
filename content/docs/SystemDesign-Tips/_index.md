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

### Caching
1. **Cache** - A piece of hardware or software that stores data, typically meant to retrieve that data faster than otherwise. 
Caches are often used to store responses to network requests as well as results of computationally tong operations.
Note that data in a cache can become stale if the main source of truth for that data (i.e., the main database behind the cache) gets updated and the cache doesn't. 

2. **Cache Hit**
When requested data is found in a cache. 

3. **Cache Miss** 
When requested data could have been found in a cache but isn't. This is typically used to refer to a negative consequence of a system failure or of a poor design choice. For example:
If a server goes down, our load balancer will have to forward requests to a new server, which will result in cache misses.

4. **Cache Eviction Policy**
The policy by which values get evicted or removed from a cache. Popular cache eviction policies include LRU (least-recently used), FIFO (first in first out), and LFU (least-frequently used). 

5. **Content Delivery Network** 
A CDN is a third-party service that acts like a cache for your servers. Sometimes, web applications can be slow for users in a particular region if your servers are located only in another region. A CDN has servers all around the world, meaning that the latency to a CDN's servers will almost always be far better than the latency to your servers. A CDN's servers are often referred to as PoPs (Points of Presence). Two of the most popular CDNs are **Cloudflare** and **Google Claud CDN**. 

### Proxies
1. **Forward Proxy** 
A server that sits between a client and servers and acts on behalf of the client, typically used to mask the client's identity (IP address), 
Note that forward proxies are often referred to as just proxies. 

2. **Reverse Proxy** 
A server that sits between clients and servers and acts on behalf of the servers, typically used for logging, load balancing, or caching. | nginx @ Pronounced "engine X°--not "N jinx”, Nginx is a very popular webserver that's often used as a reverse proxy and load balancer. Learn more: https://www.nginx.com/

3. **Load Balancer**
A type of reverse proxy that distributes traffic across servers, Load balancers can be found in many parts of a system, from the DNS layer all the way to the database layer. 
Learn more: https://www.nginx.com/

4. **Server-Selection Strategy** 
How a load balancer chooses servers when distributing traffic amongst multiple servers. Commonly used strategies include roundrobin, random selection, performance-based selection (choosing the server with the best performance metrics, like the fastest response time or the least amount of traffic), and IP-based routing. 

### Databases
1. **Relational Database** 
A type of structured database in which data is stored following a tabular format; often supports powerful querying using SQL. 
Learn more: https://www.postgresql.org/

2. **Non-Relational Database** 
In contrast with relational database (SQL databases), a type of database that is free of Imposed, tabular-like structure.  Non-relational databases are often referred to as NoSQL databases, 

3. **ACID Transaction**
    * Atomicity: The operations that constitute the transaction will either all succeed or ail fail. There is no in-between state. 
    * Consistency: The transaction cannot bring the database to an invalid state. After the transaction is committed or rolled back, the rules for each record will still apply, and all future transactions will see the effect of the transaction. Also named Strong Consistency. 
    * isolation: The execution of multiple transactions concurrently will have the same effect as if they had been executed sequentially. 
    * Durability: Any committed transaction is written to non-volatile storage. It will not be undone by a crash, power loss, or network partition. 

4. **Strong Consistency** 
Strong Consistency usually refers to the consistency of ACID transactions, as opposed to Eventual Consistency. 

5. **Eventual Consistency** 
A consistency mode which is unlike Strong Consistency. In this model, reads might return a view of the system that is stale. An eventually consistent datastore will give guarantees that the state of the database will eventuall reflect writes within a time period.

### Databases
1. **Key-Value Store** 
    A Key-Value Store is a flexible NoSQL database that's often used for caching and dynamic configuration. Popular aptions include DynamoDB, Etcd, Redis, and ZooKeeper, 
    
      * **etcd**
        Etcd is a strongly consistent and highly available key-value store that's often used to Implement leader election in a system, Learn more: https://etcd.io/ 
    
    
      * **Redis**
        An in-memory key-value store. Does offer some persistent storage options but is typically used as a really fast, best-effort caching solution, Redis is also often used to implement rate limiting. Learn more: https://redis.io/ 
    
    
      * **Zookeeper**
        Zookeeper Is a strongly consistent, highly available key-value store. It's often used to store important configuration of to perform leader election. Learn more: https://zookeeper.apache.org/ 
    

5. **Blob Storage** 
   Widely used kind of storage, in small and large scale systems. They don’t really count as databases per se, partially because they only allow the user to store and retrieve data based on the name of the blob. This is sort of like a key-value store but usually blob stores have different guarantees. They might be slower than KV stores but values can be megabytes large (or sometimes gigabytes large). Usually people use this to store things like large binaries, database snapshots, or images and other static assets that a website might have. 
   Blob storage is rather complicated to have on premise, and only giant companies like Google and Amazon have infrastructure that supports it. So usually in the context of System Design interviews you can assume that you will be able to use GCS or S3. These are blob storage services hosted by Google and Amazon respectively, that cost money depending on how much storage you use and how often you store and retrieve blobs from that storage. 
   
    * **Google Cloud Storage(GCS)** - is a blob storage service provided by Google. 
    Learn more: https://cloud.google.com/storage 
   
    * **S3** - ls a blob storage service provided by Amazon through Amazon Web Services (AWS). Learn more: https://aws.amazon.com/s3/ 

3. **Time Series Database**
   A TSDB is a special kind of database optimized for storing and analyzing time-indexed data: data points that specifically occur at a given moment in time. Examples of TSDBs are InfluxDB, Prometheus, and Graphite.

    * **influxDB** - popular open-source time series database, Learn more; https://www.influxdata.com/ 

    * **Prometheus** - A popular open source time series database, typically used for monitoring purposes. https://prometheus.io

4. **Graph Database**
   A type of database that stores data following the graph data model. Data entries in a graph database can have explicitly defined relationships, much like nodes in a graph can have edges. 
   Graph databases take advantage of their underlying graph structure to perform complex queries on deeply connected data very fast. 
   Graph databases are thus often preferred to relational databases when dealing with systems where data points naturally form a graph and have multiple tevels of relationships—for example, social networks. 

    * **Neo4j** - a popular grpah DB, consists of nodes, relationships, propreties and labels. https://neo4j.com

5. **Spatial Database** 
   A type of database optimized for storing and querying spatial data like locations on a map. Spatial databases rely on spatial indexes fike quadtrees to quickly perform spatial queries like finding all locations in the vicinity of a region. 

