---
layout: post
title: "Snowy identifiers"
date: 2016-12-05 09:55
author: scooletz
permalink: /2016/12/05/snowy-identifiers/
nocomments: true
image: /img/2016/12/stocksnap_gbrutloqum.jpg
categories: ["Architecture", "DDD", "Design"]
tags: ["architecture", "design", "identifiers"]
imported: true
---

### TL;DR

When using the [snowflake entities](http://blog.scooletz.com/2016/12/01/snowflake-entities) pattern, it's quite easy to forget about using external identifiers that we need to communicate with external systems. This post provides an easy way to address this concern.

### Identity revisited

The identifier of a snowflake entity was presented as a guid. We use an artificial non-colliding client-generated identifier to ensure, that any part of the system can generate one without validating that a specific value hasn't been used before. This enables storing different pieces of data, belonging to different contexts in different services of our system. No system leaves in vacuum though, and sometimes it requires communication with the rest of the world.

### Gate away!

A common aspect that is handled by an external system are payments. When you consider credit cards, native bank applications, PayPay, BitCoin and all the rest, providing that kind of a service on your own is not a reasonable option. That's why external services are used - the price of using one is much cheaper than delivering one. Let's stick to the payments example. How would you approach this? Would you call the external payment service from each of your services? I hope you'd not. A better approach is to create a gateway, that will act as a translator between your system and the external one.

### How many ids do I need?

Using a gateway provides a really interesting property. As the payment gateway is a part of your system, it can use the snowflake identifier. In other words, if there's an order, it's ok (under given circumstances) to use its identifier as identifier of the payment as well. Of course if you want to model these two as a part of a snowflake entity spanning across services. It'd be the payment gateway responsibility to correlate the system snowflake identifier with the external system id (integer, some string, whatever). This would create a coherent view of an entity within your system boundaries, closing the mapping in a small dedicated area of the payment gateway.

An integration with an external system closed in a small component leaving your system agnostic to this? Do we need more?

### Summary

As you can see, closing the external dependency as a gateway provides value not only by separating the interface of the external provider from your system components, but also preserves a coherent (but distributed) view of your entities.
