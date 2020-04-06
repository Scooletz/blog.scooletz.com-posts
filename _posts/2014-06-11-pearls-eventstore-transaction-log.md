---
layout: post
title: "Pearls: EventStore transaction log"
date: 2014-06-11 11:00
author: scooletz
permalink: /2014/06/11/pearls-eventstore-transaction-log/
nocomments: true
categories: ["Architecture", "Design", "Event sourcing"]
tags: ["architecture", "design", "design patterns", "EventStore", "GetEventStore", "pearls", "persistence"]
imported: true
---

I thought for a while about presenting a few projects which are in my opinion real pearls. Let's start with the [EventStore](http://geteventstore.com/ "Event Store") and one in one of its aspects: **the transaction log**.
If you're not familiar with this project, EventStore is a stream database providing complex event processing. It's oriented around streams of events, which can be easily aggregated or repartitioned with [projections](http://geteventstore.com/blog/20130212/projections-1-theory/). Based on ever appended streams and projections chasing the streams one can build a truly powerful logic around processing events.
One of the interesting aspects of EventStore is its storage engine. You can find a bit of description in [here](https://github.com/EventStore/EventStore/wiki/Architectural-Overview). ES does not abstract a storage away, the storage is a built-in part of the database itself. Let's take a look at its parts before discussing its further:

* [EventStore.Core.Services.Storage.StorageWriterService](https://github.com/EventStore/EventStore/blob/d77e195ec68efc93ab71d775319fc5608b978f7d/src/EventStore/EventStore.Core/Services/Storage/StorageWriterService.cs)
* [EventStore.Core.Services.Storage.StorageChaser](https://github.com/EventStore/EventStore/blob/d77e195ec68efc93ab71d775319fc5608b978f7d/src/EventStore/EventStore.Core/Services/Storage/StorageChaser.cs)

### Appending to the log

One the building blocks of ES is [SEDA architecture](http://en.wikipedia.org/wiki/Staged_event-driven_architecture) - the communication within db is based on publishing and consuming messages, which one can notice reviewing *StorageWriterService*. The service subscribes to multiple messages, mentioned in implementations of the *IHandle* interface. The arising question is how often does the service flushed it's messages to disk. One can notice, that method *EnqueueMessage* beside enqueuing incoming messages counts ones marked by interface *IFlushableMessage*. What is it for?

Each *Handle* method call *Flush* at its very end. Additionally, as the *EnqueueMessage* increases the counter of messages requiring flush, each *Handle* method decreases the counter when it handles a flushable message. This brings us to the conclusion that the mentioned **counter is equal 0 iff there are no more flushable messages in the queue**.

### Flushing the log

Once the *Flush* is called a condition is checked whether:
* the call was made with *force=true* (this never happens) **or**
* there are no more flush messages in the queue **or**
* the given time from the last time has passed

This provides a very powerful batching behavior. Under stress, the flush-to-be counter will be constantly greater than 0, providing flushing every given period of time. Under less stress, with no more flushables in the queue, ES will flush every message which needs to flush the log file.

### Acking the client

The final part of the processing is the acknowledgement part. The client should be informed about persisting a transaction to disk. I spent a bit of time (with help of Greg Young and James Nugent) of chasing the place where the ack is generated. It does not happen in the *StorageWriterService*. What's responsible for considering the message written then? Here comes the second part of the solution, the *StorageChaser*. In a dedicated thread, in an infinite loop, a method *ChaserIteration* is called. The method tries to read a next record from a chunk of unmanaged memory, that was ensured to be flushed by the *StorageWriterService*. Once the chaser finds CommitRecord, written when a transaction is commited, it acks the client by publishing the *StorageMessage.CommitAck* in *ProcessCommitRecord* method. The message will be translated to a client message, confirming the commit and sent back to the client.

### Sum up

One cannot deny the beauty and simplicity of this solution. One component tries to flush as fast as possible, or batches a few messages if it cannot endure the pressure. Another one waits for the position to which a file is flushed to be increased. Once it changes, it reads the record (from the in-memory chunk matched with the file on disk) processes it and sends acks. Simple and powerful.
