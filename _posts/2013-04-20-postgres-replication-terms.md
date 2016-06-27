---
layout: post
title: "PostgreSQL Replication名词解释"
description: ""
category: 开源系统
tags: [postgres, replication]
---
{% include JB/setup %}


(参考自:http://www.postgres-r.org/documentation/terms)

Replication(副本)
-----------------
For database systems, replication is the process of sharing transactional data to ensure consistency between redundant database nodes. This improves fault tolerance, which leads to better reliability of the overall system. Replications systems for databases can also be called distributed databases, especially when combined with other nice features described below. I tend to only count eager multi-master systems as distributed databases, because everything less hurts ACID principles and isn't transparent to the application.

副本是用来在事务处理上保持不同数据节点之间一致性的过程(?), 副本可以提高故障容忍度, 提高整体系统的可靠性. 

Load Balancing(负载均衡)
------------------------
Replication is often coupled with load balancing to improve read performance. Some replication solutions have an integrated load balancer, which knows about the underlying system. Others rely on OS dependent or third party load balancers.

负载均衡应该没有什么可说的.

Replication Methods
-------------------
Databases can be kept coherent in many different ways. A very common and simple approach is statement based replication, where SQL statements are distributed between the nodes. Non-deterministic functions, like now() or random(), pose some problems for that method. Another very common method is log shipping. Unfortunately the database system's log is often not meant as an exchange format for replication and thus it's hard to do replication of only parts of a database. Thus some replication solutions have their own binary format which is specifically designed for replication.

介绍了一些Replication的情况下会出现问题的函数, 例如now(), random()等.

Divergence(散度, 可以作为衡量节点之间一致性的指标->类似ACID中的一致性C? 各种一致性)
-----------------------------------------------------------------------------------
Keeping data coherent across multiple nodes is quite expensive in terms of network latency. Thus many systems try to avoid network delays by allowing the nodes to diverge slightly, meaning they allow conflicting transactions to commit. To revert to a coherent and consistent database, those conflicts need to be resolved, either automatically or manually. Such conflicts violate the ACID property of the database system, so the application needs to be aware of that.

保持数据一致非常昂贵, 很多系统为了避免网络延迟通过允许轻度的分散, 也就是允许不一致的事务被提交, 这些冲突可以通过自动或者手动解决, 会违反ACID的属性, 需要引起注意.

Synchronous vs Asynchronous Replication(同步或者异步副本)
---------------------------------------------------------
According to (Wikipedia's definition of synchronization), database replication is considered data synchronization, as opposed to process synchronization. However, internally a database systems has to synchronize locks or at least transactions, which would clearly be considered process synchronization.

Within the context of database replication, the most common definition of synchronous replication is, that as soon as a transaction is confirmed to be committed, all nodes of a cluster must have committed the transaction as well. This is very expensive in terms of latency and amount of messages to be sent, but it prevents divergence. In asynchronous replication systems, other nodes can apply the transactional data at any later point, thus the nodes may serve different, possibly even conflicting snapshots of the database.

同步副本要求集群中的所有节点在事务提交时, 都进行该事务的提交. 该操作在时延和消息发送上非常昂贵, 但是可以降低散度. 而异步的副本系统允许不同节点的事务延迟提交, 但是可能会导致不一致的数据库镜像.

Eager vs Lazy Replication(主动复制和被动复制)
---------------------------------------------
The terms eager and lazy are more often used in the literature about database replication. Sometimes synonymously to synchronous or asynchronous replication. Other times, there's a nifty difference: instead of stating, that all nodes need to process transactions synchronously, it's often sufficient to state that the nodes or replicas are eventually kept coherent. This means that transactions must be applied in the very same order, but not necessarily synchronously. According to that definition, eager replication is somewhere between sync and async: it allows nodes to lag behind others while still preventing divergence.

Another way to think about this distinction is, that eager systems have to replicate the transactional data before confirming the commit, while lazy systems replicate that data at some time after committing, so conflicts can arise.

Lazy replication is always asynchronous and does not prevent divergence. Due to this tiny but important difference, I prefer the term lazy to asynchronous. In any case however, the process of exchanging data to detect and resolve conflicts is called reconciliation. How often to reconciliate is a tradeoff between the allowed lag time and the load for reconciliation. It's also important to know, that lazy systems may reduce the probability of divergence by reconciliating more often, but the possibility of divergence cannot be eliminated completely (otherwise the system would be eager).

主动复制介于同步和非同步之间复制, 被动复制属于非同步复制. 主动复制可以避免分散: 1) 同步复制不会导致分散, 2) 异步复制但是其事务的提交顺序与同步一直, 从理论上也不会造成分散; 被动复制不会避免分散: 被动复制解决数据冲突的过程叫做reconciliation(调和), 系统进行reconciliation的频率会提高一致性, 但是不能避免分散.

Distributed Querying(分布式查询)
--------------------------------
Distributed querying allows a single query to use multiple servers. This improves performance for long running read-only transactions. Short and simple queries should better be answered from a single node. The Postgres documentation speaks of "Multi-Server Parallel Query Execution".

分布式查询允许一个查询运行在多个服务器上, 对于long running的事务, 会能提高性能, 而小事务, 单个节点会更快. 详见官方文档:

http://www.postgresql.org/docs/8.2/static/high-availability.html

Multi-Server Parallel Query Execution

Data Partitioning(数据分区)
---------------------------
A common distinction is vertical vs horizontal partitioning. Vertical partitioning splits into multiple tables with fewer columns, much like during the process of normalization. When partitioning horizontaly, the database systems stores different rows in multiple different tables.

In a multi-master system both methods can be used to split data across multiple nodes, so that every node only holds parts of the complete database. This obviously decreases reliability, as fewer nodes store the same tuple. And it requires some sort of distributed querying to be able to reach all data a transaction needs. But it certainly improves total capacity of your cluster and reduces the total load for writing transactions.

普遍的分区方式有垂直分割和水平分割, 垂直分割将一张表切分为多张列数比较少的表(列存是否为一种极端的垂直分割); 水平分割则是将数据库的数据中不同行分布到不同表中(http://www.postgresql.org/docs/9.2/static/ddl-partitioning.html)

Shared-disk vs. Shared-nothing Clusters(共享磁盘和无共享的集群)
---------------------------------------------------------------
A shared-disk cluster describes a bunch of nodes which share a common disk-subsystem, but with otherwise independent hardware. While that term makes sense, "shared-nothing" is often confusing people. It means that the nodes of a cluster share nothing, not even the disks. Of course, both types are commonly connected via some type of packet switching network. While shared-nothing clusters mostly consist of commodity hardware, shared-disk clusters often use specialized and expensive storage systems.

Shared-disk systems can provide are a good base for single-master replication solutions with failover capability. Using a clustered filesystem in a shared-nothing environment can be an inexpensive alternative.

It's a common misconception, that a shared-disk cluster would allow a faster eager multi-master replication systems than a shared-nothing one. The reasoning being, that less data needs to be transferred. But that's no where the problem is, because it's not the network throughput, but the latency that matters. In other words: doing conflict detection using CPU, memory and a network is faster than doing it using CPU, memory and shared disks.

共享磁盘需要额外的存储系统, 无共享的集群基本使用常见的硬件即可. In other words: doing conflict detection using CPU, memory and a network is faster than doing it using CPU, memory and shared disks.(无共享的集群更快?)

Clustering(集群)
----------------
While clustering is a very popular term, it does not have a well defined meaning with regard to database systems. Lots of different techniques, like replication, load balancing, distributed querying, etc.. are called clustering here and there.

这个属于营销词汇, 只要不是单机就叫做集群, 没啥具体定义.

Grids
-----
With regard to database replication, the very same applies for "grids", only potentiated. It's a plain marketing term, IMO.

呵呵, 市场营销词汇(看来严谨的做研究不能看报道, 那些个都是广告)