---
layout: post
title: "Orchestrating processes for fun and profit"
date: 2017-06-01 08:55
author: scooletz
permalink: /2017/06/01/orchestrating-processes-for-fun-and-profit/
nocomments: true
image: /img/2017/05/stocksnap_wsg2ii60up.jpg
categories: ["Business", "C#"]
tags: ["async", "orchestration", "saga"]
imported: true
---

### TL;DR

I've already shown [here](https://blog.scooletz.com/2017/04/13/orchestrating-your-processes-with-durable-task/) that with some trickery you can write orchestrations in C# based on *async-await*. It's time to revisit this idea, now with a custom orchestration of mine.

### Show me the code!

The orchestration is based on the event sourcing infrastructure built by me. This project is not public (yet) but it's similar to any other ES library. The main idea behind building an orchestration on top of it is that state changes of an orchestration are easily captured as events like: *GuidGenerated*, *CallRecorded, UtcDateTimeObtained*, etc etc. If this is the case, we could model any orchestration as an event-sourced aggregate.

Here it is an example of reserving a trip, that needs to coordinate between reserving a hotel and a flight. Additionally a few calls were added to just show what the orchestration is capable of

![orchestration.png](/img/2017/05/orchestration.png)

* Line 19: orchestration can delay it's execution. It does not mean that it will stay in memory for that long. It will record the need of delay to be woken up when the time comes
* Line 22: when a Guid is generated with *Orchestration.NewGuid* it means that it will have the same value when the orchestration is executed again. Process dies and the orchestration is executed elsewhere? Don't you worry, the guid value has been recorded
* Line 25: same with dates
* Line 28: an external call to any system. Here, we try to *ReserveHotel*. 3 attempts are made with a exponential back off. Between calls, if the configured timespan is long enough, saga can be swapped from memory (like with *Delay*)
* Line 33: same as with hotel
* Line 38: the compensation part

### Execute it!

The execution would take the persisted events and replay them if process failed, machine died, etc. on another host. Because before every external call, all the recorded events are persisted, it ensures that even during a rerun, all the values will be exactly the same and calls that were already made won't be made again.

### Summary

Orchestrating with *async-await* trickery is much easier and can be written in a dense, simple way of a regular method call. Having an event-sourced foundation makes it even easier as you can use already existing pieces and persist all orchestration events as event of some kind of aggregate.
