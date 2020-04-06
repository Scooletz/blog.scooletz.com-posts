---
layout: post
title: "Event sourcing: making it functional (3)"
date: 2017-01-12 09:55
author: scooletz
permalink: /2017/01/12/event-sourcing-making-it-functional-3/
image: /img/2016/12/3.jpg
categories: ["DDD", "Event sourcing"]
tags: ["domain driven design", "event sourcing"]
imported: true
---

### TL;DR

We're on our journey to move from event sourcing oriented on the aggregates to a more functional approach. In [the first entry](http://blog.scooletz.com/2017/01/05/event-sourcing-making-it-functional-1), we went through some of the DDD building blocks. In the second, we defined an interface of the aggregate that we'll work with. Before, going to the event sourced approached, let's go through the aggregate's implementation, somehow related to the Blue Book approach, without event sourcing.

### All entries in this series:

* [1 - back to foundations](https://blog.scooletz.com/2017/01/05/event-sourcing-making-it-functional-1/)
* [2 - aggregate sketch](http://blog.scooletz.com/2017/01/09/event-sourcing-making-it-functional-2/)
* 3 - first implementation
* [4 - finding events](http://blog.scooletz.com/2017/01/16/event-sourcing-making-it-functional-4/)
* [5 - applying events](http://blog.scooletz.com/2017/01/19/event-sourcing-making-it-functional-5/)
* [6 - recording aggregate](http://blog.scooletz.com/2017/01/23/event-sourcing-making-it-functional-6/)
* [7 - the fully functional end](http://blog.scooletz.com/2017/01/26/event-sourcing-making-it-functional-7/)

### Payment aggregate!

Let's dive straight into the code

![paymentapi](/img/2016/12/paymentapi.png)

We can see that a payment is issued for a user, with a specified payment method and a given amount of money. Yes, it's simplistic, but bear with me, and follow its implementation for a while.

The other method is responsible for receiving the gateway response and applying it onto the aggregate. The status and the description are updated accordingly.

So far so good? In the next entry we'll make this aggregate event sourced!
