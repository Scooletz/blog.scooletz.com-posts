---
layout: post
title: "Converging processes"
date: 2017-01-30 09:55
author: scooletz
permalink: /2017/01/30/converging-processes/
nocomments: true
image: /img/2017/01/stocksnap_yb5jncv2kq.jpg
categories: ["Design", "Uncategorized"]
tags: ["design", "modelling", "process manager"]
imported: true
---

### TL;DR

Completing an order is not an easy process. A payment gateway may not accept payments for some time. A coffee can be spilled all over the book you ordered and a new one needs to be taken from a storage. Failures may occur in different moments of the pipeline, but still your ordered is received. Is it a one process or multiple? If one, does anyone takes every single piece into consideration? How to ensure that eventually a client will get their product shipped?

### Process managers and sagas

There are two terms used for these processors/processes. They are **process managers** and **sagas**. I don't want to go through the differences and marketing behind using one or another. I want you to focus on a process that handles an order and that reacts to three events:
* PaymentTimedOut - occurring when the response for Payment was not delivered before the specific timeout
* PaymentReceived - the payment was received in time
* OrderCancelled - the order for which we requested this payment was cancelled

What messages will this process receive and in which order? Before answering, take into consideration:

* the scheduling system that is used to dispatch timeouts,
* the external gateway system (that has its own SLA),
* messaging infrastructure

What's the order then? Is there a predefined one? For sure there isn't.

### Convergence

Your aim is to provide a convergent process. You should review all the permutations of the process inputs. Being given 3 types of input, you should consider 3! = 6 possibilities. That's why building a few processes instead of one is easier. You can think of them as sub-state machines that later, can be composed into a bigger whole. This isn't only a code complexity, but a real mental overhead that you need to design against.

### Summary

Designing multiple small processes as sub-state machines is easier. When dealing with many events to react to, try to extract smaller state machines and aggregate them on a higher level.
