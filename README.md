# System Design

[Course resources](https://www.youtube.com/playlist?list=PLMCXHnjXnTnvo6alSjVkgxV-VH6EPyvoX)

Course author: Gaurav Sen

#### Course 1: System Design Basics: Horizontal vs. Vertical Scaling

- Scalability:
  - Scalability refers to the ability of the server to further expand. Generally speaking, in order to achieve better performance of the server, there are often two ways to expand: i) Buy more machines and let these machines serve as servers, so that when multiple requests come, the server will process more efficiently ; ii) Buy a larger machine, so that the processing capacity and processing speed of the server will be higher.
  - Usually, increasing the number of machines is called horizontal scalability; increasing the number of machines is called vertical scalability.

![horizontal_vs_vertical](./imgs/horizontal_vs_vertical.png)

-Horizontal vs. Vertical Scaling

| | HORIZONTAL | VERTICAL |
| -------- | ----------------------------------------- --------- | ----------------------------------- |
| Load Balancing Required | N/A |
| An error occurred | Resilient (resilient, a machine error, the system will not end) | Single point a failure |
| Process Communication | Network calls (RPC, Remote Process Communication) | IPC, Inter process Communication |
| Consistent | DATA INCONSISTENCY | Consistent |
| Scalability | SCALES WELL AS USERS INCREASE | Hardware limit |



#### Course 2. System Design Primer: How to start with distributed systems?

- think about to have a pizza shop
  - how to scaling?
    - 1 [Vertical Scaling, scalability] Optimize process and increase throughput with the same resource.
    - 2 Preparing before hand at non-peak hours.
    - 3【Backups】keep a backup and avoid single point of failure.
    - 4【Horizontal Scaling】Hire more resources.
    - 5 Micro Service architecture
    - ![micro_service_arch](./imgs/micro_service_arch.png)
    - 6 Distributed System (Build a new system)
    - 7 Load Balance (Build a manager to send request to Server in a balanced way.)
    - 8 Decoupling 【Decoupling】
    - 9 Logging and Metrics calculation.
    - 10 Extensible [Scalability] refers to the ability to add components and expand functions


#### Course3.4. Consistent Hashing

Consistent Hashing helps to make Load Balancing

- Why use consistent hashing?
  - When hashing the request, if the same user gets the same result, then his request will be sent to the same server, so the user information can be stored in the server's cache;
  - If different servers are accessed, user information will be continuously replaced from the cache, and the complexity of accessing information will increase.
- Clockwise hash structure
  - ![hash_arch](./imgs/hash_arch.png)
  - Advantages: If there are enough servers, the probability of each request being assigned to a server is 1/N on average;
  - Disadvantage: If there are few servers, the load will be unbalanced (increase or decrease the server, the load will be unbalanced)
- improved structure
  - Each server uses multiple hash functions. In this way, although there are fewer servers, there will be more after mapping; after one server is reduced, it will also be uniformly reduced in the circle structure, so as to keep some resources unchanged as much as possible .
  - ![hash_arch_v2](./imgs/hash_arch_v2.png)
  - Classic pie chart (minimal changes)
- scenes to be used?
  - Load balancing issues in distributed systems
  - Web cache (the same user is mapped to the same server after consistent hashing, which can ensure efficient cache hit)
  - database



#### Course 5. Message queue

- Notifier with: Load Balancing, Heart-Beat and DB Persistence --> **Message/Task Queue**
- ![message_queue_intro](./imgs/message_queue_intro.png)
  - Understanding: The request to be processed by the server is put into the task queue, and then all tasks are evenly distributed to all servers. When some servers are down, the notifier detects the server ID through the heartbeat packet, and persists The unfinished task (request) corresponding to the sid is found in the optimized db, and then the unfinished request is redistributed to other servers in a load-balanced manner. Finally realize asynchronous RPC.
- Examples: JMS(Java Message Service), ActiveMQ, RabbitMQ
- Usage scenarios: business decoupling/ultimate consistency/broadcasting/off-peak flow control; if you need strong consistency and focus on the processing results of business logic, then you only need RPC.



#### Course 6. What is a micro-service architecture and it's advantages?

- monolithic vs. microservice

  - Monolithic application: The service is composed of multiple modules, and all modules are packaged into one package and run as one process. The user interface, data access layer, and data deployment layer are tightly coupled. [Example: each module of the system may access the same db, so they are collaborative]

    - [Advantages] Good for small team: If the team is relatively small and everyone knows the business process of the system, then there is no need to make microservices, and the individual is only responsible for the development of each module.
    - Less Complex: There are few things to consider, such as how to divide the system into several parts, and how to communicate between each part, ensure consistency, and so on. [personal understanding]
    - Less Duplication: For other types of files such as configuration files, there is no need to copy them to each service, only one copy is enough.
    - Faster, Procedure Calls: The collaboration between modules is only as an in-process collaboration, and can communicate without RPC and other methods.
    - [Disadvantages] More context required: When a new team member joins, he needs to understand too many things. He needs to understand the whole system to know what work he needs to do (otherwise he will not understand)
    - Complicated Deployments: Deployment will be more complicated. When you check that there is a problem with a module, you need to repackage the entire project and deploy it again.
    -Single Point of Failure! : After a problem occurs at a single point, the entire service needs to be restarted.

  - Microservices: A set of loosely coupled components that work together and perform tasks. [Each module has its own db]

    -【Advantages】Scalability
    - Easier for new team members
    - Working in Parallel
    - Easier to reason about: For example, a chat system can easily know which microservice is important, so expand more machines for that service to improve performance
    - [Disadvantages] Hard to Design, Needs skilled Architects: For example, if you find that S1 always interacts with S2, then you can combine them into a Service

    

#### Course 7. What is Database Sharding?【Sharding】

-Horizontal Partitioning vs. Vertical Partitioning
  - ![db_partition](./imgs/db_partition.png)
  - vertical split
    - Group the tables in the database (not necessarily different tables, but also different columns of the same table) according to business, etc., and put the same type in the same new database.
  - Split horizontally
    - If a table is too large, divide it into multiple groups according to certain rules (such as a certain value>threshold; hash value, etc.), and store these groups in different databases.
  - ![db_horizontal_vertical](./imgs/db_horizontal_vertical.png)
  - Sub-database sub-table (that is, table-level split and database-level split)
  - General business is relatively small and sub-table can be used, and the level can be sub-database; at the same time, problems caused by sub-database should also be paid attention to, such as common distributed problems, distributed transactions, high availability, data consistency, etc.
- Split the database instance horizontally -- database sharding
  - 【Advantage】
    -Help the horizontal scaling of the system (horizontal scaling): After horizontal scaling, migrate some data to new nodes
    - Accelerated query response time
    - Reduce the impact of downtime: After sharding, downtime only affects the system of the database corresponding to the slice
  - [Disadvantage]
    - The sharding database architecture is very complex: reflected in two aspects, i) the sharding process may cause data loss or table damage, which is very risky; ii) the correct use of sharding may also have a significant impact on the team's workflow (although The division is based on certain rules, but when a certain business does not follow the rules to access data, the disadvantage of fragmentation is reflected).
    - Fragmentation can easily lead to imbalance, and the data may be skewed towards a sliced ​​database, and finally answer to the maximum load, facing the problem of fragmentation again
    - Once a database is sharded, it is difficult to restore it to an unsharded schema
    - Not all database engines support sharding
  - **Key Based Sharding** Key Based Sharding
    - [Description] After hashing a static column (data that does not change over time), write it into different databases according to the value.
    - [Disadvantages] When adding/deleting servers dynamically, things will become tricky, resulting in data remapping and migration, and a consistent hash method can be used to minimize the amount of data migration
    - [Advantage] Evenly distribute data
  - **Range Based Sharding** Range Based Sharding
    - [desc] For example, sharding according to the price of a certain commodity
    - [ad] Easy to implement, easy to write and read
    - [disad] It cannot prevent the uneven distribution of data. For example, there are many products with a price of 50-100 yuan, but few products with a price of more than 100,000 yuan
  - **Directory Based Sharding** Directory Based Sharding
    - [desc] A certain column of the original data is used as the partition key, and the partition of the data is realized by maintaining a lookup table. That is to say, when looking up data, now find the partition where the data is located according to the shard key, and then search in the partition
    - ![db_direct_based_sharding](./imgs/db_direct_based_sharding.png)
    - [Advantages] Flexibility. Compared with range sharding, only the range of keys can be specified, and key sharding can only specify a hash function, which is more troublesome to change.
    - [Disadvantage] A lookup table is required for each lookup, and the performance is degraded; there is a single point of failure, and the lookup table is damaged, which will cause the system to fail to operate normally
- Picture reference in this chapter: https://zhuanlan.zhihu.com/p/57185574



#### Course 8 How Netflix onboards new context: Video Processing at scale

-Problem Description
-Video formats and resolutions
  - codec to compress the video, high quality, medium quality, low quality. . .
  - Different resolutions, 1080p, 720p, etc.
- Chunk processing  
  - Divide video into chunks
  - Scenes>Timestamp, divide the video into blocks by scene, not by time: divide the video into a shot according to 4s, and combine multiple shots into a scene
  - Sparse & Dense Movie, corresponding to different request strategies, the number of videos obtained for each request
-Storage
  - Amazon S3, which only saves constant data and does not provide modification
- Open Connect for video caching
  - Created Open Connect Box to cache certain videos and avoid access to US servers
-Summary



#### Course 10. What is Distributed Caching? Explained with Redis!

> Caching in distributed systems is an important aspect of designing scalable systems. We start by discussing what caching is and why we use it. We then discuss what are the key properties of caches in distributed systems.
>
> The cache management strategy of LRU and Sliding Window is mentioned here. For high performance, the cache eviction strategy must be chosen carefully. In order to keep data consistent and memory usage low, we must choose write through or write back consistency strategy.
>
> Cache management is important as it relates to cache hit ratio and performance. We discuss various scenarios in a distributed environment.

-Why use Caches?
  - Save Network Calls: Repeated request results are saved in the cache
  - Avoid Computations: Keep calculation results in cache
  - Reduce DB Load: Save some DB information in the cache to reduce the load on the database
  - [The first two cases are to speed up the server's response to the client]
-Cache Policy
  - LRU (Least Recently Used)
  - sliding window
- Disadvantages of cache strategy failure (choose a bad cache strategy
  -Extra Call
  - Thrashing (continuously replacing unused data)
  - Consistency

-In-memory Cache & Global Cache
  - in-mem cache: faster; more compact
  - Global cache: more accurate; features of independent scalability
-Write-Through and Write-Back Cache
  - write through: When writing, first update the cache content, and then update the db
    - When multiple servers have written content X in their caches, this strategy can only update one cache
  - write back: When writing, first write to db, and then write the updated content of db to cache
  - It is necessary to choose a cache consistency strategy based on the importance of the data.
    - For non-important data, use the write-through strategy, write to the cache first, and then write to db after a period of time; although there may be wrong data for other servers, when other servers miss the cache, they will be updated again (also That is to say, this kind of non-important data, don’t care if it is right)



#### Course 15. What is the Publisher Subscriber Model?

> Microservices benefit from loose data coupling, which is provided by the publish-subscribe model. In this model, events are generated by publishing services and consumed by downstream services.
>
> Designing microservice interactions involves event handling and consistency checking. We studied the publish-subscribe architecture to evaluate its advantages and disadvantages compared to the request-response architecture.
>
> This type of architecture relies on message queues to ensure event delivery. An example is rabbitMQ or Kafka. This architecture is common in real life scenarios and interviews.
>
> The event model is fine to use in microservices if the transactions do not have strong consistency guarantees. Here are the main advantages:
>
> 1) Decouple the services of the system.
> 2) Easily add subscribers and publishers without notifying each other.
> 3) Turn multiple points of failure into single points of failure.
> 4) Interaction logic can be moved to service/message broker.
>
> Disadvantages:
> 1) The extra layer of interaction slows down the service
> 2) Cannot be used in systems that require strong data consistency
> 3) Additional cost for the team to redesign, learn and maintain message queues.
>
> The model provides the basis for event-driven systems.

- Traditional microservice architecture
  - failure latency: When a service is waiting for other microservices, if the waiting service fails, it will take a long time for the waiting service to know
  - data inconsistent
- Publish-subscribe model
  - Send messages to brokers, like RabbitMQ, Kafka
  - ![C15_Pub_Sub_Model](./imgs/C15_Pub_Sub_Model.png)
- Model advantages:
  -Decoupling
  - Simplifies Interactions:
    - S1 only needs to send a request to the broker, instead of sending it to S0 or S2 separately, the interaction is more convenient;
    - Convert multi-point failure to single-point failure
  - Some Transaction Capability: Once S1's request is sent to the broker, it will persist, so it will ensure that S0 and S2 will receive the request.
  - Easily Scalable: Newly added services only need to be registered with the message broker
- Disadvantages:
  -poor consistency(major drawback)
  - [Idempotency] The results of one or more requests initiated by the user for the same operation are consistent. For example, if a user initiates a payment request and fails due to network reasons, the next request should not deduct money multiple times
  -Idempotency still required
  - RPC will reduce efficiency



#### Course 16: Why do Database fail? AntiPatterns to avoid!

> Databases are commonly used to store various types of information, but one situation where it becomes a problem is when used as a message broker.
>
> Databases are seldom designed to handle message-passing functionality, and thus are not a good replacement for specialized message queues. This pattern is considered an anti-pattern when designing systems.
>
> The following are possible disadvantages:
> 1) The polling interval must be set correctly. Too long will make the system inefficient. Too short will subject the database to a heavy read load.
> 2) DB with heavy read and write operations. Usually, they are good at one of the two.
> 3) To write a manual delete program to delete read messages.
> 4) Difficult to scale both conceptually and physically.
>
> Disadvantages of message queues:
> 1) Add more moving parts to the system.
> 2) The cost of setting up MQ and training is significant.
> 3) Might be overkill for small services.
>
> In a system design interview, it is important to be able to reason why or why a system does not need a message queue. These reasons allow us to debate the pros and cons of the two approaches.
>
> However, there are blogs out there explaining why databases work perfectly as message queues too. A solid understanding of the pros and cons helps assess their effectiveness for a given scenario.
>
> In general, for small applications, databases are fine because they don't introduce additional moving parts to the system. For complex messaging needs, it is useful to have an abstraction such as a message queue that handles messaging for us.

[background] Using db as mq is considered an anti-pattern and needs to be avoided!

- Database as Message Queues
  - The database can only receive information from the client, but cannot send information to the client; therefore, the client (subscriber) needs to use polling to determine whether it has its own message; the problem lies in frequent polling -> you need to consider the load of the DB / Long interval (inefficient) reply, in short need to balance the polling interval.
  - For MQ, it is often faced with heavy read (R) or write (create, write, delete) tasks; but db is always not good at both; and multi-server access to db also faces system levels such as locks and deadlocks The problem
  - Poor scalability; for a system that is already fully loaded, adding an additional server means that another db is needed to implement mq when horizontally expanding, but communication between dbs becomes difficult.
- The advantage of message queue is to make up for the above points; for example, mq will send messages to the server, so it does not have the problem of server polling time; mq only needs to focus on writing performance and ignore reading performance; mq can be expanded horizontally by expanding memory
- Disadvantages of mq:
  - There is no need to use mq for small services, db can do the job well
  - Pay attention to the structure of the system. If there are few write operations in the system, but a lot of reads, then you need to consider the db and optimize the read performance of the db
  - The training cost of mq is high
