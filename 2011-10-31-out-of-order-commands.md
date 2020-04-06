---
layout: post
title: "Out of order commands"
date: 2011-10-31 11:00
author: scooletz
permalink: /2011/10/31/out-of-order-commands/
categories: ["Architecture", "DDD"]
tags: ["architecture", "design", "idempotent"]
imported: true
---

In the previous posts a simple mechanism of storing information needed for operation idempotence was introduced. A simple hash table, which state is transactionally saved with the state of object onto which the send operation was applied. How about receiving operations out of order? What if infrastructure (for instance, messaging system) will pass one operation earlier than the second, which in reality occurred earlier?

*It's time to make it explicit and start calling elements in the DDD manner. So for sake of reference, the object considered as the subject of an operation is an aggregate root. The operation is of course a message. The modeling assumes using the event sourcing as a storage for aggregates' states.*

Assume, that the aggregate, which the command is sent to, has a property called *Version*, incremented with each event applied on. Assume then, each command contains a version number, which is supposed to be equal to the aggregate's version. If, during dispatch, these two values are different, an exception is thrown and command do not change the state of the aggregate. It's a simple optimistic concurrency implementation, allowing discarding out of order commands sent to an object.

To make it more interesting, consider a sharded system, where specific aggregates are stored by different nodes (but for each aggregate there is one node where it is stored). An aggregate's events (state changes) have to be propagated across all the nodes/shards in the same idempotent manner as commands are sent to aggregates. It's easy to apply hashtable for each node and with using the very same key: aggregateId with version but it would mean storing *all* the pairs of aggregate identifiers with their versions, which could possibly bring down each of your nodes (or make you use GBs of memory). Can the trivial fact, that version is increased with every event on the aggregate, could be used for some optimization? You'll see in the next entry.
