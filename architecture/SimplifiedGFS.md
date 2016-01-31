# What and why
In this post, I will talk about Google File System, notably a distributed file system. So what's a distributed file system? I don't want to go to the very detail. Instead I'd like to describe some scenarios in which the solution may involve a distributed file system.

## Taobao CDN
Taobao is an online marketing system. There are tons of pictures which will be displayed in web pages. Generally these pictures are served by CDN. So what if a client uploads a new picture? How others see it immediately? One simple solution is file synchronization(e.g. rsync). However, rsync is not scalable. So we need a large filesystem with shared access across the network. The distributed file system comes to rescue.

## Batch processing
Say you are running Facebook. There are tons of services in Facebook which produce huge amount of log files everyday. We need to analyze these logs later. So how do we store these logs? Normal file system is not sufficient, we need some system with large storage capacity! Yeah, that's the distributed file system.

# A Simplified GFS Design
I will present a simplified distributed file system design based on the GFS paper, and I will concentrate on the storage engine, not the data model it provides. GFS is quite complex which involves a lot of design aspects. Basically it has the goals:

+ Data-intensive: big data oriented
+ Scalability
+ Availability & Reliablity
+ Performance

I will not talk about everything in the GFS paper, that's a rather complex job. I don't like an extremely long article, so I would concentrate on scalability, availability, and reliability.

## Scalability
So how to scale a system? Generally there are two kind of nodes in a distributed system. One is stateless computation node and the other is stateful storage node. Scaling stateless nodes is very simply, simple add more nodes. But scaling stateful nodes is the issue. Basically different stateful services have difference way to scale, but they both keep to be a complicated job!

In our case, our storage engine provides raw block storage, but you can of course build more sophisticated data model such as files, blobs, key-values on top of it. We have N servers for storage. We call them **DataServers**. When we have more blocks to store, we can simply add more DataServers to our system. For example:

+ DataServerA: Block1, Block2, Block3
+ DataServerB: Block4
+ DataServerC: Block5, Block6

If we want to scale out our system,  just more Dataservers and more blocks:

+ DataServerD: Block7
+ DataServerE: Block8, Block9

Then the question follows. How does a client reference Block8? How does it know which DataServer a block resides? GFS utilizes a centralized fashion. We setup a **Master** which retains all blocks' metadata.

Using a master makes the system easy to implement and easy to reach consistency.

***You can simple view DataServers as ROOMS, and blocks as PACKAGES. If we have more rooms, we can store more packages! But first of all, we must know how to find a package since there are so many rooms!***

### Addressing the block: Master
The **Master** is comprised of 2 services:

+ NameManager: manage filenames and blocks and their relationship.
    + Retain filename namespace
    + Retain block namespace
    + Map a filename to a block id
+ DataServerManager: manage DataServers
    + Retain a list of DataServer metainfo
    + Keep heartbeats with DataServers

When a client requests for a block, it sends Master a filename, NameManager will map it to a block id. DataServerManager will ask all DataServers who has the block via rpc.

For better performance, Master will cache block-to-dataserver map. But it won't be persistent(via commit log). Only NameManager and DataServerManager persist. When Master crashed and restarted, all metadata retained by NameServer and DataServerManager will be recovered.

### Storing blocks: DataServer
DataServer provides DataRead and DataWrite services. Clients use block id to refer to a block, and read from or write to that block.

## Availability & Reliablity
Two of the most important properties of a distributed system are availability and reliability. So how does our system be available and reliable?

### HA of Master
Master is a single-point of failure. To make such node reliable, we have two methods:

+ Quick recovery:
    + When the node crashes, a monitor detects it and restarts it immediately.
    + This requires that the node must be started in a few seconds.
+ Failover:
    + Master have a slave, Master will forwards all updates to slave. Slave will be synced with Master
    + A monitor keep monitoring both master and slave.
    + The monitor will notify the slave when master failure was detected

### HA of DataServer
Basically, we can do the same things to make DataServers reliable and available as mentioned above. But that's too expensive. Rather than replicating DataServers, we replicates blocks.

So who replicates blocks across DataServers? Master!

Master will periodically scan all blocks. When one block has not enough replicas. Master will pick a source DataServer which contains the block, and a target DataServer based on its workload. And send a request to issue the replication task.

Another question is that, one block have few copies. When we read or write, which copy should we go?

+ Read: client side load balance. Reads can go every copy
+ Write:
    + Only one copy can accept write requests, we call it **MasterBlock**
    + Other copies are **SlaveBlock**
    + MasterBlock accepts writes and syncs with SlaveBlocks
    + In GFS, we say that MasterBlock have the **LEASE**. When a client wants to write to a block, Master must pick a MasterBlock from that block copies and give it the lease. **LEASE** also facilitates fault tolerance.

## Quick Implementation
Lots of details are not mentioned above. I will describe them briefly here.

+ Master Persistence: CommitLog + Snapshot, this is almost a standard
+ Block Deletion: how to delete a block?
    + A Client sends a delete msg
    + NameManager removes the block metainfo
    + DataServerManager notifies all DataServers containing that block
+ New DataServer Attendance
    + New DataServer sends heartbeat to Master, DataServerManager add it
+ Rebalancing blocks
    + During block scanning, pick a overload DataServer and a underload DataServer
    + Pick a block from the overload and move it to the underload
+ Etc

# Reference
+ [Annotated TFS](https://github.com/qqiangwu/annotated-tfs)
+ [Elements of scale composing and scaling data platforms](http://highscalability.com/blog/2015/5/4/elements-of-scale-composing-and-scaling-data-platforms.html)
+ [Taobao Xun Shen](http://blog.sina.com.cn/s/articlelist_1765738567_0_1.html)
