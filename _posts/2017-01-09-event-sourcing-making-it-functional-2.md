---
layout: post
title: "Event sourcing: making it functional (2)"
date: 2017-01-09 09:55
author: scooletz
permalink: /2017/01/09/event-sourcing-making-it-functional-2/
nocomments: true
image: /img/2016/12/number-437918_1280.jpg
categories: ["DDD", "Event sourcing"]
tags: ["domain driven design", "event sourcing"]
imported: true
---

### TL;DR

In the last post we covered the basic concept of DDD, [the aggregate and its root](http://blog.scooletz.com/2017/01/05/event-sourcing-making-it-functional-1). Before we move on with the non-functional approach, let's try to define at least one aggregate, that could be used as an example for future posts and code samples.

### All entries in this series:

* [1 - back to foundations](https://blog.scooletz.com/2017/01/05/event-sourcing-making-it-functional-1/)
* 2 - aggregate sketch
* [3 - first implementation](http://blog.scooletz.com/2017/01/12/event-sourcing-making-it-functional-3/)
* [4 - finding events](http://blog.scooletz.com/2017/01/16/event-sourcing-making-it-functional-4/)
* [5 - applying events](http://blog.scooletz.com/2017/01/19/event-sourcing-making-it-functional-5/)
* [6 - recording aggregate](http://blog.scooletz.com/2017/01/23/event-sourcing-making-it-functional-6/)
* [7 - the fully functional end](http://blog.scooletz.com/2017/01/26/event-sourcing-making-it-functional-7/)

### A payment

For sake of providing an aggregate example let me reuse the payment example I introduced for [the service kata purposes](http://blog.scooletz.com/2016/12/08/code-kata-with-business-rules). The aggregate boundary is handling a single payment. This includes:

* selecting payment method
* registering an answer from the payment gateway
* storing status

Having this we can easily imagine that the API of this aggregate could be similar to:

![paymentapi](/img/2017/01/paymentapi1.png)

As you can see, there are no dummy setters or meaningless primitive properties. We exposed meaningful methods and properties for showing the part of the state we want to show. Additionally, the payment constructor shows all the parameters that are required to issue a payment.

With this example we can move forward and provide more insight into its initial implementation and refactoring it towards more functional design.
