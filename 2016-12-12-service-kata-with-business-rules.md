---
layout: post
title: "Service kata with Business Rules"
date: 2016-12-12 09:55
author: scooletz
permalink: /2016/12/12/service-kata-with-business-rules/
nocomments: true
image: /img/2016/12/stocksnap_ihueia0y8t.jpg
categories: ["Architecture", "DDD", "Design"]
tags: ["architecture", "design"]
imported: true
---

### TL;DR

In [the previous post](http://blog.scooletz.com/2016/12/08/code-kata-with-business-rules/) we started working on a code kata and discovered that instead of creating a new monolithic giant we could tackle the complexity of a process by modelling it right in its natural boundaries: contexts. It this post we continue this journey.

### Requesting payment

Let's spend some time on modelling a process of ordering a membership. It's been said that it requires a payment and as soon as the payment is done, the membership is activated. We introduced the **PaymentReceived** event as an asynchronous response to the payment request.

Consider a membership order with the following identifier

11112222-3333-4444-5555-666677778888

When accepting the request for a membership, **Membership** sends a request to the **Payments** with the following information

![payment_request](/img/2016/12/payment_request.png)

It is important to see, that the caller generates identifier, which has following properties:

* In this case it reuses it's own identifier for a different context to use [snowy identifiers](http://blog.scooletz.com/2016/12/05/snowy-identifiers/) to create [snowflake entities](http://blog.scooletz.com/2016/12/01/snowflake-entities/)
* As the caller generates id and stores it, in case of the failure when requesting a payment, it can be POSTed again as it's idempotent (any http status indicating that it already exists means that the previous call was accepted and processed).

Using approach in a service oriented architecture enables idempotence (everyone knows the id upfront).

### Events strike back

The result of the payment, after receiving money is an event **PaymentReceived** which is published to all the interested parties. One of them will be **Membership**, which would simply take the paymentId and check if there's an order for a membership with the same identifier. It there is, it can be checked as paid. Simple and easy. The same will apply to other rules in other contexts.

There's really no point of making this **ONE BIG APP TO RULE THEM ALL**. You can separate services according to business units and design towards integrating them.

Again, depending on the used tools, you can have events delivered by a bus to all the subscribers all use ATOM feed to publish events in there, and consume them by polling from other services.

### **Summary**

These two posts show that raising modelling questions is important and that it can help to reuse existing structures and applications in creating new robust systems. They do not cover transactions, retries and more. You can use tools that solve it for you like a messaging bus or you'll need to handle it on your own. Whatever path you choose, the modelling techniques will be generally the same and you can use them to bring real value into the existing ecosystem **instead of creating the single new shiny application that will rule them all**.
