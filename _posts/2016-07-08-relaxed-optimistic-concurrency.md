---
layout: post
title: "Relaxed Optimistic Concurrency"
date: 2016-07-08 09:00
author: scooletz
permalink: /2016/07/08/relaxed-optimistic-concurrency/
nocomments: true
categories: ["Design"]
tags: ["DDD", "design", "modelling"]
imported: true
---

**TL;DR**

When using the optimistic concurrency approach for entities that are updated frequently, some of the actions may fail because of the conflicting version numbers. A proper modelling technique distilling if business requirements can be loosened may greatly increase the chances of succeeding with commands issued against these entities improving overall performance of an application and a lowering a probability of errors.

### Optimistic Concurrency

The optimistic concurrency is an approach for ensuring non overlapping updates over a given entity. It's supported by the majority of heavy ORMs and applied simply by adding a conditional where at the end of the update. For example

[code lang="sql"]
UPDATE Orders
-- more updated columns
SET version = @version + 1
WHERE id = @id AND version = @version

```

This approach ensures, that if any other operation updated the entity in the meantime, this update will fail. Additionally, if an ORM is capable of counting rows that should have been updated, like NHibernate does, it can abort a transaction and throw an exception informing that some of the operations that were planned to be executed failed.

The optimistic concurrency approach is not a unique SQL technique. It's popular in many NoSql databases like Azure Table Storage for example. When updating an entity, its ETag is added as the *If-Match* header, ensuring, that if the entity was modified after retrieval and updated, the operation, again, will fail. See Update operation documentation [here](https://msdn.microsoft.com/en-us/library/azure/dd179427.aspx).

Finally, when applying Domain Driven Design and operating on an Aggregate Root, this technique is the easiest one to ensure, that the aggregate root is truly a transaction boundary. If the root has its version updated with every change of the aggregate, then two concurrent operations cannot be executed and one will fail, still, preserving the root as a transaction boundary. This applies to aggregate roots, no matter if you immerse them into Event Sourcing or a regular ORM mapped graph of entities. Just update the root with every operation and your aggregate will be just fine.

As it's been shown above, optimistic concurrency is a simple and powerful tool that in a world of NoSql and transactional-boundaries-got-right may be the only one to ensure atomicity of operations.

### Limitations

When using optimistic concurrency, the flow of applying a change is a bit different. Instead of just updating a property, or a value, the following approach is taken

1. An aggregate is retrieved with its version
1. If the state allows it, a command is executed
1. The aggregates' state is updated conditionally (if the version is unchanged)

Again, this ensures that the updated is applied on the version that a business logic operated onto, but limits the concurrent access.

For services using Event Sourcing, instead of retrieving entity all of the events are retrieved and a state of an aggregate is rebuilt. If [snapshots](http://blog.jonathanoliver.com/event-sourcing-and-snapshots/) are used, only events with versions bigger than a snapshot must be retrieved. If the snapshot is preserved in a in memory cache, then possibly, no events will be retrieved if the snapshot's version is equal to the number of aggregate's events so far. Events that are a result of a command are appended to the store conditionally. Depending on the storage it can be the stream version when using EventStore [AppendToStreamAsync](https://github.com/EventStore/EventStore/blob/release-v3.8.0/src/EventStore.ClientAPI/IEventStoreConnection.cs#L78) or update of a root markup entity when using a custom relational store.

### An example

Let's consider an example of a GitHub-like issue. Every issue has an option of locking it. It can be used for instance to lock an issue created by a troll (you don't feed the troll) and disallow adding more comments. For sake of argument:

1. let's model all comments as a part of the issue aggregate (as always, there are many models that can be applied)
1. optimistic concurrency is used for all commands.

A business requirement for locking an issue could look like:

*when an issue is locked no user should be able to add more comments*

It's quite common, that when seeing a requirement like this, developers don't ask questions. It's even more unfortunate, that some companies require to *just follow the analysis*. Let's try to relax this requirement a little bit by asking some questions:

1. *Is it required to lock the issue immediately?*
1. *Could an issue be considered locked after some short period of time (less than 1s) after locking it?*
1. *Could we allow adding some comments during this period?*

If the answers point towards no need of an immediate lock, there's a space to handle locking in a relaxed manner

### Relaxed Optimistic Concurrency

If an operation can have its preconditions relaxed and can be performed after achieving some state it can be executed with much less friction. In the previous example, the state when a user can add a comment is a created issue. The precondition is a *non-locked* issue, but it's ok to add a comment to a locked issue within some time boundaries. Consider the following flow

1. An aggregate is retrieved with its version
1. If the state allows it, a command is executed
1. The aggregates' state is updated <del>conditionally (if the version is unchanged)</del> **appending** **the change unconditionally**

Depending on the storage and the applied design in can be done in many ways.

When using Event Sourcing with EventStore a special version can be passed to the appending method which represents [any version](https://github.com/EventStore/EventStore/blob/release-v3.8.0/src/EventStore.ClientAPI/ExpectedVersion.cs#L28). This appends events unconditionally. This means that a locking operation and adding a comment can be done in parallel without conflicts!

When using a relational database, an issue entity can be retrieved to check it's state. Next, a comment entity can be added separately, without updating the version of the issue itself. Again, because adding a comment does not change the version, the friction on the aggregate is lowered.

### Summing up

Don't take requirements for granted, but rather ask for the reasoning behind them. Try to relax requirements for areas which may suffer from the high contention. The model is just a model. There are no true or false models but these which help you or make your work harder. Choose wisely :)
