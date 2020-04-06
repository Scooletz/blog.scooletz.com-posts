---
layout: post
title: "Process manager in event sourcing"
date: 2014-11-21 11:00
author: scooletz
permalink: /2014/11/21/process-manager-in-event-sourcing/
categories: ["DDD", "Event sourcing"]
tags: ["event sourcing", "idempotent", "process manager"]
imported: true
---

There is a pattern which can be used to orchestrate collaboration of different aggregates, even if they are located in different contexts/domains. This patters is called a [process manager](http://kellabyte.com/2012/05/30/clarifying-the-saga-pattern/). What it does is handling events which may result in actions on different aggregates. How can one create a process manager handling events from different sources? How to design storage for a process manager?

In the latest take of event sourcing I used a very same direction taken by [EventStore](http://geteventstore.com/). My first condition was to have a single natural number describing the sequence number for each event committed in the given context/domain (represented as module). This, because of using an auto-incrementing identity in a relational database table, even when some event may be rolled back by transaction, has resulted in an monotonically increasing position number for any event appended in the given context/domain. This lets you to use the number of a last read event as a cursor for reading the events forward. Now you can imagine, that having multiple services results in having multiple logs with the property of monotonically increasing positions, for example:

* Orders: e1, e2, e3, e6
* Notifications: e1, e2, e3, e4

If a process manager reads from multiple contexts/domains, you can easily come to a conclusion that all you need to store is a last value of a cursor from the given domain. Once an event is dispatched, in terms of finishing handling by the process manager, the cursor value for the context event was created within is updated. This creates an easy but powerful tool for creating process managers with *at-least-once* process guarantee for all the events they have to process.

If a process provides guarantee of processing events *at-least-once* and can perform actions on aggregates, it may, as action on aggregate and saving the state of a PM isn't transactional, perform the given action more than once. It's easy: just imagine crushing the machine after the action but before marking the event as dispatched. How can we ensure that no event will result in a doubled action? This will be the topic of the next post.
