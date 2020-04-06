---
layout: post
title: "TOP 1 method to lower SLA of your service"
date: 2018-11-28 09:55
author: scooletz
permalink: /2018/11/28/top-1-method-to-lower-sla-of-your-apps/
image: /img/2018/11/app.jpg
categories: ["Architecture", "Azure", "Cloud", "Design"]
tags: ["azure", "cloud", "design"]
imported: true
---

*The title is not a mistake. This post is about the best technique to lower SLA of your services. Hopefully, you won't be using this technique (unless building a non-reliable service is in your contract).*

In the cloud, it really easy to take a dependency on just-another-service (TM) provided by the vendor. You need to search for something? Just use the search service. Need some storage? Just add the storage service. Need to publish a message? Just use this messaging. Need to publish a bit more? Throw in another messaging.

Hopefully you can see the pattern above. The most important question to ask is:

> How adding another dependency impacts your SLA?

### SLA impact

Every single piece of any cloud has a magic value called SLA, which stands for Service Level Agreement. Usually, it's presented as a percentage of time, when the service is up. Depending on the vendor, when a service goes down, you can get a discount, an apologizing email or nothing. If you wanted to describe SLA in other words, that would be a probability of having the service up. If it's a probability, and every single service is independent on others, it has an interesting property.

There's a way to calculate a probability of independent events happening at the same time. In these case, it would be: having all the services running. Let's consider services A, B and C that your application is dependent on with following probabilities of them running:

* P(A) = 0.99
* P(B) = 0.98
* P(C) = 0.97

Let's calculate the result, the upper boundary of having your application running

P(your app) = P(A) * P(B) * P(C) = 0.99 * 0.98 * 0.97 = 0.941094

Yes. The the upper boundary of SLA for your service is only 94.1094% This excludes all the bugs, exception and unpredictable errors of your application. We can clearly see, that taking *just-another-dependency (TM)*  which is required for your application to run, **lowers its** upper boundary of **SLA**.

We know the way how to make the application fragile then. How about making it less fragile?

### Getting less fragile

You could argue, that as the application is hosted somewhere it will always be bounded by the host SLA. To put it simply, something needs to respond to all the HTTP requests. It's even more probable that another dependency be required: a database or a cache (because cache would solve a potential db problem, introducing its own can of worms).

Is there a way to make it more robust? Are we always doomed to multiply these probabilities and lower the SLA of our apps.

### No app is better than any app

What you could do, is to design your application or a service in a way, that the critical path does not include the application at all. One solution, would be using a messaging infrastructure that simply accepts the request and stores it till the application is up. Hopefully, the application will be able to pick it up as soon as possible, but if not, nothing is lost. The only part that the caller is dependent onto is the messaging part.

In modern environments like Azure, even when using the simplest messaging provided by it, you can easily provide the write-only access (see, my article about [not calling Azure Functions at all](http://blog.scooletz.com/2018/03/19/serverless-more-secure-sas-tokens/)). The caller is able just to add a message, with no permissions for querying, managing, etc. Simple as that. Is it any better?

It is. For the caller, there's a single SLA that they should consider, and this is not provided by you or your team but by the infrastructure. I'd bet that it's much more likely that the cloud based infra will have much better SLA than your service (just take into consideration, that your app is built on top of this infrastructure - hence, multiplication of SLAs can be applied).

### Summary

If you want to build fragile systems, use the first part of this article and stack up as many dependencies as you can. If not, remove the unnecessary dependencies, and, if possible, remove your application or service from the call path at all, leaving the call for the infrastructure.
