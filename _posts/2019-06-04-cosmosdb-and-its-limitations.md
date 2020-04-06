---
layout: post
title: "CosmosDB and its limitations"
date: 2019-06-04 08:55
author: scooletz
permalink: /2019/06/04/cosmosdb-and-its-limitations/
image: /img/2019/06/cosmos-1.jpg
whitebackgroundimage: true
categories: ["Architecture", "Azure", "Cloud", "Design"]
tags: ["architecture", "azure", "CosmosDB"]
imported: true
---

*It's great when you can discuss advantages of a product. The even more important part to know is to know its limitations. What is possible and what is not. This post is not a rant but a short summary of a few limitations found during my recent investigation of CosmosDB.*

### Partitions

The partitioning is not mentioned that often when describing CosmosDB as a planet scale database. If you're familiar with the stateful part of the Service Fabric or Azure Storage Services, you probably heard about partitions. To enable scaling, data are partitioned between multiple shards and cannot be updated in cross-partition transactions.

In Azure Storage Services, Tables, you needed to explicitly provide the Partition Key and the Row Key that identifies a row within partition. In Service Fabric, you'd access the partition and then work on data inside of it. With CosmosDB, when using it's core API, you just select a path in your JSON document, that will be accessed and treated as the Partition Key. The premise is that two documents with the same Partition Key are placed in the same logical partition, but multiple logical partitions can be placed in a single Physical Partition.

Every partition is replicated. If a database doesn't have the multi-master writes enabled, for every partition there will be a single leader and several followers. A really good description of the replication and its architecture can be found in [Global data distribution with Azure Cosmos DB - under the hood](https://docs.microsoft.com/en-us/azure/cosmos-db/global-dist-under-the-hood).

### Transactions

The first and foremost design principle behind transactions could be summarized as

> Limit the transaction lifetime

Whenever is an option for having a long-running (wallclock time) transaction, the answer from CosmosDB is *nope*. Let's go through several cases:

1. Can you update a document atomically?

    Yes. A document resides within a single partition. It can be easily retrieved from the storage and have its ETag verified. You can definitely update a document atomically.

1. Can I update multiple document from different partitions?

    *Nope*. Updating documents in different partitions would result in running a 2 Phase Commit protocol. This is a heavy tool. Cosmos does not like to get heavy.

1. Can I run an interactive transaction on the client side, to fetch some data, run some logic and then fetch more and finally update?

    *Nope*. Running a transaction on the client side would require to have it last for a long period of time. Each request goes to the db and back, which results in more delay and keeping the snapshot of data for even longer (CosmosDB uses Snapshot Isolation).

1. Can I share a transaction, like I did in the past with ORMs, when implementing auditing etc.?

    *Nope*, at least not directly. The stored procedure is executed atomically and so far I've found no way to augment it. What you could try to do is to use triggers that are run in the same manner, to do some "additional work".

### Stored procedures

How one could address some of the limitations in regards to working on multiple documents and introduce some conditional flow in it if needed? If we can't bring data to the computation as it involves a lot of latency, maybe we could bring the computation to the data?

This is how CosmosDB addresses the lack of the client side interactive transactions. If you want to run a transaction between multiple documents, you need to write a stored procedure that will be run by CosmosDB. To make this happen, you provide a snippet of javascript and register it in the database. Why Javascript you say? The reason for this is that CosmosDB runs [the Chakra javascript engine](https://github.com/microsoft/ChakraCore) on every node to enable hosting and executing the operations. For sure there are some question to be answered:

1. Can I run the stored procedure for an infinite time period?

    *Nope*. You can't do it. The stored procedure is run in the Leader of a specific partition. Its time is precious! You don't want to use it for counting *sin(x)* or something similar. Additionally, once the execution is started, CosmosDB will start a Snapshot transaction. Limiting the time of the snapshot is crucial to do not keep the old data in the hot set.

1. Can CosmosDB inform me about breaching the execution time?

    Yes. CosmosDB returns a bool value from its js methods. You can construct a flow, that will quickly abort when there's no more budget for your stored procedure.

1. Can I access and update multiple documents?

    Yes, but only within a single partition. Remember, stored procedures are run on Leader of a specific partition. It can access its data and that's all.

### Do I need to care about it?

It's up to you. If you store "just some documents", probably you don't need to. If you think about CosmosDB as a primary store for GBs or TBs of data, especially, when there are some heavy processes that work on these data, definitely you should spend some time on learning about CosmosDB design choices.

It's better to know limitations of the technology you use. If you don't, it will show you yours.
