---
layout: post
title: "Aggregate, an idempotent receiver"
date: 2014-12-02 11:00
author: scooletz
permalink: /2014/12/02/aggregate-an-idempotent-receiver/
nocomments: true
categories: ["DDD", "Event sourcing"]
tags: ["EAI", "event sourcing", "integration patterns"]
imported: true
---

In [the previous post](http://blog.scooletz.com/2014/11/21/process-manager-in-event-sourcing/) I covered the process manager subscribing to and consuming events from multiple sources. Additionally, it was show that saving the position of read logs after performing action is sufficient to get at-least-once delivery (retry in case of errors).

Let me consider an aggregate which an action is invoked on. As the only transactional boundary that can be used is the aggregate itself, to each call from process manager we'll add additional data:

1. hash (unique, [SHA1](http://en.wikipedia.org/wiki/SHA-1) probably) of the process manager identifier and the name of the origin module where the handled event was taken from
1. the order number of the handled event

This two values combined in an event, will allow in one transaction to check, whether the action has been already applied and skip it if needed. Everything in one transaction.
As order numbers for the given hash can only increase, the state of this idempotent received can be modeled as a dictionary with Sha1 value as its key and the order number as its value.
The only disadvantage is additional event added to the aggregate for each action performed within a process manager. Fortunately, a scavenging process, a similar one to [this from EventStore](https://github.com/EventStore/EventStore/wiki/Architectural-Overview#scavenging). When events are dumped to a file from a store of your choice, only the last value for the given Sha1 hash can be stored.
