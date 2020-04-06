---
layout: post
title: "Event sourcing: making it functional (4)"
date: 2017-01-16 09:55
author: scooletz
permalink: /2017/01/16/event-sourcing-making-it-functional-4/
image: /img/2016/12/4.jpg
categories: ["DDD", "Event sourcing"]
tags: ["domain driven design", "event sourcing"]
imported: true
---

### TL;DR

In the last entry we defined the aggregate implementation that we'll work on. Let's move forward and make it an event sourced aggregate.

### All entries in this series:

* [1 - back to foundations](https://blog.scooletz.com/2017/01/05/event-sourcing-making-it-functional-1/)
* [2 - aggregate sketch](http://blog.scooletz.com/2017/01/09/event-sourcing-making-it-functional-2/)
* [3 - first implementation](http://blog.scooletz.com/2017/01/12/event-sourcing-making-it-functional-3/)
* 4 - finding events
* [5 - applying events](http://blog.scooletz.com/2017/01/19/event-sourcing-making-it-functional-5/)
* [6 - recording aggregate](http://blog.scooletz.com/2017/01/23/event-sourcing-making-it-functional-6/)
* [7 - the fully functional end](http://blog.scooletz.com/2017/01/26/event-sourcing-making-it-functional-7/)

### Events

The first and the most important event, is the fact of issuing the payment itself. Let's define it in the following way:

![paymentissued](/img/2017/01/paymentissued.png)

The event contains all the data provided to the constructor of the payment aggregate in the past.

The next one is raised and applied when the payment is processed successfully. Let's make it a class without any members. We just want to record a notion of success.

![paymentissued](/img/2017/01/paymentissued1.png)

The last but not least is the one raised in case of the error. We make the errors explicit in here as we'd like to react to payments failure. One could model it in a different way, providing one event class, but then again, just follow this take on the payment problem.

![paymentissued](/img/2017/01/paymentissued2.png)

### Summary

We discovered three meaningful events:

1. PaymentIssued
1. PaymentProcessedSuccessfully
1. PaymentProcessedWithFailure

In the next entry we will start rewriting the aggregate to use these.
