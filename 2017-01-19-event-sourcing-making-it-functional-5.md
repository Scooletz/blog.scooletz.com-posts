---
layout: post
title: "Event sourcing: making it functional (5)"
date: 2017-01-19 09:55
author: scooletz
permalink: /2017/01/19/event-sourcing-making-it-functional-5/
categories: ["DDD", "Event sourcing"]
tags: ["domain driven design", "event sourcing"]
imported: true
---

### TL;DR

After [defining events of the Payment aggregate](http://blog.scooletz.com/2017/01/16/event-sourcing-making-it-functional-4) it's time to move on and work on applying these events.

### All entries in this series:

* [1 - back to foundations](https://blog.scooletz.com/2017/01/05/event-sourcing-making-it-functional-1/)
* [2 - aggregate sketch](http://blog.scooletz.com/2017/01/09/event-sourcing-making-it-functional-2/)
* [3 - first implementation](http://blog.scooletz.com/2017/01/12/event-sourcing-making-it-functional-3/)
* [4 - finding events](http://blog.scooletz.com/2017/01/16/event-sourcing-making-it-functional-4/)
* 5 - applying events
* [6 - recording aggregate](http://blog.scooletz.com/2017/01/23/event-sourcing-making-it-functional-6/)
* [7 - the fully functional end](http://blog.scooletz.com/2017/01/26/event-sourcing-making-it-functional-7/)

### State

To apply events, the state of Payment will be extracted to a separate class. This should be a familiar pattern to all event sourcing practitioners, but let me show the code for clear understanding of our approach

![paymentstate](/img/2017/01/paymentstate.png)

### How to construct the state

1. There are no public setters. The state has readonly properties.
1. All events are applied with an *Apply* method.

1. The only way to change the state is to apply an event.
1. No *Apply* throws an exception. The event has already happened and the state is just an accumulator of the changes.

### New Payment Aggregate

The Payment aggregate now can be transformed to use this state, by removing all the state changes and raising/applying events in these places:

![paymentstate](/img/2017/01/paymentstate1.png)

### Summary

We know how to extract a state from the aggregate and apply all the events. Although Payment uses events, it does not store them in any form nor allows accessing them for processing. In the current shape the Payment isn't much different from the previous version. We extracted a few event classes and the state, but we still can't treat events as the first class citizen. we need to move one step further and we will do it in the next blog post.
