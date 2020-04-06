---
layout: post
title: "Event sourcing: making it functional (7)"
date: 2017-01-26 09:55
author: scooletz
permalink: /2017/01/26/event-sourcing-making-it-functional-7/
categories: ["DDD", "Event sourcing"]
tags: ["domain driven design", "event sourcing"]
imported: true
---

### TL;DR

It's the seventh chapter of making event sourcing functional. In [the last post](http://blog.scooletz.com/2017/01/23/event-sourcing-making-it-functional-6) we introduced the base class for aggregates capturing events and applying them on the state. In this entry we question this choice and make the final step to make our code more functional. Let's go!

### All entries in this series:

* [1 - back to foundations](https://blog.scooletz.com/2017/01/05/event-sourcing-making-it-functional-1/)
* [2 - aggregate sketch](http://blog.scooletz.com/2017/01/09/event-sourcing-making-it-functional-2/)
* [3 - first implementation](http://blog.scooletz.com/2017/01/12/event-sourcing-making-it-functional-3/)
* [4 - finding events](http://blog.scooletz.com/2017/01/16/event-sourcing-making-it-functional-4/)
* [5 - applying events](http://blog.scooletz.com/2017/01/19/event-sourcing-making-it-functional-5/)
* [6 - recording aggregate](http://blog.scooletz.com/2017/01/23/event-sourcing-making-it-functional-6/)
* 7 - the fully functional end

### Question your choices

Let's again take a look at the method responsible for receiving the gateway response

![paymentstate](/img/2017/01/paymentstate3.png)

This is a void method. It's not a pure function though. It "returns" its result by passing emitted events to the apply function. We could rewrite it in the following manner:

![paymentstate](/img/2017/01/paymentstate4.png)

Now you can probably see, that there are two methods in there. One that emits, the event, possibly taking the state into consideration and the second that applies the event. We can make the first one even more generic and make it return *IEnumerable* of events.

![PaymentState.png](/img/2017/01/paymentstate5.png)

### Final revelation

Now you can see, that *ReceiveImpl* is the true implementation of the aggregate action, but it does not require the aggregate class at all! It gets the state, the action parameter and returns events! The *ReceiveGatewayResponse* is now just an infrastructure code that applies events, which is totally unneeded! We no longer have the Payment aggregate! All we have is just a set of functions that acts on the state basis, accepts some parameters and returns events. We can make it even an extension to call it in an easier way.

![paymentstate](/img/2017/01/paymentstate6.png)

This pure function is so easy to test. You can test it using Given, When, Then with events, but you can test it with regular unit testing as well.

Now you can see that we were able to split the *Payment* aggregate into set of functions, that accept a state and other needed parameters returning a enumerable of events. Is there anything easier to test and to work with? For sure you need to pay some tax by introducing storing and applying events on the infrastructure side, but still it's worth it, as we change aggregates, from being classes to just sets of simple functions.

### Summary

I hope you enjoyed this series and that I was able to encourage you to see aggregates and event sourcing from a bit more functional point of view. All the commenters, thank you for providing valuable feedback. See you soon!
