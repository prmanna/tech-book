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

### No-SQL Databases
1. **Key-Value Store** 
    A Key-Value Store is a flexible NoSQL database that's often used for caching and dynamic configuration. Popular aptions include DynamoDB, Etcd, Redis, and ZooKeeper, 
    
      * **etcd**
        Etcd is a strongly consistent and highly available key-value store that's often used to Implement leader election in a system, Learn more: https://etcd.io/ 
    
    
      * **Redis**
        An in-memory key-value store. Does offer some persistent storage options but is typically used as a really fast, best-effort caching solution, Redis is also often used to implement rate limiting. Learn more: https://redis.io/ 
    
    
      * **Zookeeper**
        Zookeeper Is a strongly consistent, highly available key-value store. It's often used to store important configuration of to perform leader election. Learn more: https://zookeeper.apache.org/ 
    
      * **DynamoDB**
        An key-value store by AWS, this provides eventual consistency.
    
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

### Replication & Shrading

1. **Replication** - The act of duplicating the data from one database server to others. This is sometimes used to increase the redundancy of your system and tolerate regional failures for instance. Other times you can use replication to move data closer to your clients, thus decreasing latency of accessing specific data.

2. **Sharding** - Sometimes called data partitioning, sharding is the act of splitting a database into two or more pieces called shards and is typically done to increase the throughput of your database. Popular sharding strategies include: 
    * Sharding based on a client's region. 
    * Sharding based on the type of data being stored (e.g: user data gets stored in one shard, payments data gets stored In another shard) 
    * Sharding based on the hash of a column (only for structured data)  

### Peer-To-Peer Networks
1. Peer-To-Peer Network
A collection of machines referred to as peers that divide a workload between themselves to presumably complete the workload faster than would otherwise be possible. Peer-to-peer networks are often used in file-distribution systems. 

2. Gossip Protocol 
When a set of machines talk to each other in a uncoordinated manner in a cluster to spread information through a system without requiring a central source of data. - ad ° 


### Rate Limiting
1. **Rate Limiting** 
    The act of limiting the number of requests sent to or from a system. Rate limiting is most often used to limit the number of incoming requests in order to prevent DoS attacks and can be enforced at the IP-address level, at the user-account level, or at the region level, for example. Rate limiting can also be implemented in tiers; for instance, a type of network request could be limited to 1 per second, 5 per 10 seconds, and 10 per minute. 

2. **DoS Attack** 
    Short for “denial-of-service attack”, a DoS attack is an attack in which a malicious user tries to bring down or damage a system in order to render it unavailable to users. Much of the time, it consists of flooding it with traffic. Some DoS attacks are easily preventable with rate limiting, while others can be far trickier to defend against. 

3. **DDoS Attack** 

   Short for “distributed denial-of-service attack", a DDoS attack is a DoS attack in which the traffic flooding the target system comes from many different sources (like thousands of machines), making It much harder to defend against. 

4. **Redis** 
An in-memory key-value store. Does offer some persistent storage options but Is typically used as a really fast, best-effort caching solution, Redis is also often used to implement rate limiting. Learn more: https://redis.io/ 


### Publish/Subscribe Pattern

1. **Publish/Subscribe Pattern** 
Often shortened as Pub/Sub, the Publish/Subscribe pattern Is a popular messaging model that consists of publishers and subscribers. Publishers publish messages to special topics (sometimes called channels) without caring about or even knowing who will read those messages, and subscribers subscribe to topics and read messages coming through those topics. 
Pub/Sub systems often come with very powerful guarantees like at-least-once delivery, persistent storage, ordering of messages, and replayability of messages. 

2. **Apache Kafka** 
A distributed messaging system created by Linkedin. Very useful when using the streaming paradigm as opposed to polling. Learn more: https://kafka.apache.org/

3. **Cloud pub/sub**
A highly-scalable Pub/Sub messaging service created by Google, Guarantees at-least-once delivery of messages and supports “rewinding” in order to reprocess messages. 
Learn more: https://cloud.google.com/pubsub/ 


### MapReduce
1. **MapReduce** 
A popular framework for processing very large datasets in a distributed setting efficiently, quickly, and in a fault-tolerant manner. A MapReduce job is comprised of 3 main steps: 
    * the Map step, which runs a map function on the various chunks of the dataset and transforms these chunks into intermediate key-value pairs. 
    * the Shuffle step, which reorganizes the intermediate key-value pairs such that pairs of the same key are routed to the same machine in the final step. 
    * the Reduce step, which runs a reduce function on the newly shuffled key-value pairs and transforms them into more meaningful data. The canonical example of a MapReduce use case is counting the number of occurrences of words in a large text file. When dealing with a MapReduce library, engineers and/or systems administrators only need to worry about the map and reduce functions, as well as their inputs and outputs. All other concerns, including the parallelization of tasks and the fault-tolerance of the MapReduce job, are abstracted away and taken care of by the MapReduce implementation. 

2. **Distributed File System** 
A Distributed Ale System is an abstraction over a (usually large) cluster of machines that allows them to act like one large file system. The two most popular implementations of a DFS are the Google File System (GFS) and the Hadoop Distributed File System (HDFS). Typically, DFSs take care of the classic availability and replication guarantees that can be tricky to obtain in a distributed-system setting, The overarching idea is that files are split into chunks of a certain size (4MB or 64MB, for instance), and those chunks are sharded across a large cluster of machines. A central control plane is in charge of deciding where each chunk resides, routing reads to the right nodes, and handling communication between machines, Olfferent DFS implementations have slightly different APIs and semantics, but they achieve the same common goal: extremely largescale persistent storage,
3. **Hadoop** 
A popular, open-source framework that supports MapReduce jobs and many other kinds of data-processing pipelines. Its central component is HDFS (Hadoop Distributed File System), on top of which other technologies have been developed. Learn more: https://hadoop.apache.org/ 


### Security And HTTPS

1. **Symmetric Encryption** 
    A type of encryption that relies on only a single key to both encrypt and decrypt data. The key must be known to all parties involved in the communication and must therefore typically be shared between the parties at one point or another. 
    Symmetric-key algorithms tend to be faster than their asymmetric counterparts. 
    The most widely used symmetric-key algorithms are part of the Advanced Encryption Standard (AES). 

2. **Asymmetric Encryption**
    Also known as public-key encryption, asymmetric encryption relies on two keys—a public key and a private key—to encrypt and decrypt data. The keys are generated using cryptographic algorithms and are mathematically connected such that data encrypted with the public key can only be decrypted with the private key. 
    While the private key must be kept secure to maintain the fidelity of this encryption paradigm, the public key can be openly shared. 
    Asymmetric-key algorithms tend to be slower than their symmetric counterparts. 

3. **AES** 
    Stands for Advanced Encryption Standard. AES is a widely used encryption standard that has three symmetric-key algorithms (AES-128, AES-192, and AES-256). 
    Of note, AES is considered to be the "gold standard" in encryption and is even used by the U.S. National Security Agency to encrypt top secret information. . 

4. **HTTPS** 
    The HyperText Transfer Protocol Secure is an extension of HTTP that's used for secure communication online. It requires servers to have trusted certificates (usually SSL certificates) and uses the Transport Layer Security (TLS), a security protocol built on top of TCP, to encrypt data communicated between a client and a server. 
    { TLs The Transport Layer Security is a security protocol over which HTTP runs in order to achieve secure communication online. "HTTP over TLS" Is also known as HTTPS. 

5. **SSL Certificate** 
    A digital certificate granted to a server by a certificate authority. Contains the server's public key, to be used as part of the TLS handshake process in an HTTPS connection. 
    An SSL certificate effectively confirms that a public key belongs to the server claiming it belongs to them. SSL certificates are a crucial defense against man-in-the-middie attacks, 

6. **Certificate Authority** 
    A trusted entity that signs digital certificates—namely, SSL certificates that are relied on in HTTPS connections. 

7. **TLS Handshake** 
   The process through which a client and a server communicating over HTTPS exchange encryption-related information and establish a secure communication. The typical steps in a TLS handshake are roughly as follows: 

    * The client sends a client hello string of random bytes—to the server. 
    * The server responds with a server hello another string of random bytes—as well as its SSL certificate, which contains its publle key. 
    * The client verifies that the certificate was issued by a certificate authority and sends a premaster secret—yet another string of random bytes, this time encrypted with the server's public key—to the server. 
    * The client and the server use the client hello, the server helio, and the premaster secret to then generate the same symmetric encryption session keys, to be used to encrypt and decrypt all data communicated during the remainder of the connection. 

