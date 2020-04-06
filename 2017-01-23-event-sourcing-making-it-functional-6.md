---
layout: post
title: "Event sourcing: making it functional (6)"
date: 2017-01-23 09:55
author: scooletz
permalink: /2017/01/23/event-sourcing-making-it-functional-6/
categories: ["DDD", "Event sourcing"]
tags: ["domain driven design", "event sourcing"]
imported: true
---

### TL;DR

In [the last entry](http://blog.scooletz.com/2017/01/19/event-sourcing-making-it-functional-5) we changed the Payment aggregate to modify state by raising events. The events weren't captured though. In this post, we'll change the aggregate to enable recording of these changes.

### All entries in this series:

* [1 - back to foundations](https://blog.scooletz.com/2017/01/05/event-sourcing-making-it-functional-1/)
* [2 - aggregate sketch](http://blog.scooletz.com/2017/01/09/event-sourcing-making-it-functional-2/)
* [3 - first implementation](http://blog.scooletz.com/2017/01/12/event-sourcing-making-it-functional-3/)
* [4 - finding events](http://blog.scooletz.com/2017/01/16/event-sourcing-making-it-functional-4/)
* [5 - applying events](http://blog.scooletz.com/2017/01/19/event-sourcing-making-it-functional-5/)
* 6 - recording aggregate
* [7 - the fully functional end](http://blog.scooletz.com/2017/01/26/event-sourcing-making-it-functional-7/)

### Capturing events

The easiest way to capture events is to make them pass one additional method. We could provide one Apply method in the aggregate itself that beside applying the event it would. To make it usable in all the aggregates we could create a base class that would have this method. Let's apply it to the Payment first

![paymentstate](/img/2017/01/paymentstate2.png)

Now, we need the aggregate base class to hold the capture the applied events.

### The base aggregate

We use the base class to track the changes and provide one point of entry to applying events. The *dynamic* is used to implement it fast. If you wanted to optimize, you could remove it by writing a custom dispatcher that calls directly a method from the derived aggregate.

![paymentissued.png](/img/2017/01/paymentissued5.png)

When the action ends, a handler executing it calls *GetEvents* and gets all the events that were raised during this session. This enables you to write simple tests with a Given-When-Then approach and easily extract the changes for persistence purposes. On the other hand, we introduced the base class for the aggregate. What could be done more?

### Summary

By introducing a base **AggregateRoot** class we enabled capturing all raised events. This simplified a little bit Payment aggregate itself, but introduced the base class. It's time to move to the final part where we make it functional removing some code we've written so far.
