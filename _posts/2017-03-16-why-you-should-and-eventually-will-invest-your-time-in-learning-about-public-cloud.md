---
layout: post
title: "Why you should and eventually will invest your time in learning about public cloud?"
date: 2017-03-16 08:55
author: scooletz
permalink: /2017/03/16/why-you-should-and-eventually-will-invest-your-time-in-learning-about-public-cloud/
image: /img/2017/03/scooletz_image3.jpg
categories: ["Azure"]
tags: ["azure", "cloud"]
imported: true
---

### TL;DR

Within 2-5 years the majority of applications will be moved to public cloud. You'd better be prepared for it.

### Economies of scale

You might have heard that economy of scale does not work for software. Unfortunately, this is not the case for public cloud sector. It's cheaper to buy 1000000 processors than to buy one. It's cheaper to buy 1000000 disks than to buy one. It's better to resell them as a service to the end customer. And that's what public cloud vendors do.

### Average app

The majority of applications does not require fancy processing, or 1ms service time. They require handling peaks, being mostly available and costing no money when nobody uses one. I'd say, that within 2-5 years we will all see majority of them moving to the cloud. If there is a margin, where the service proves its value and it costs more than its execution in the cloud, eventually, it will be migrated or it will die with a big IT department running through the datacenter trying to optimize the costs and make ends meet.

### Pure execution

The pure execution has arrived and its called Azure Functions (or Lambda if you use the other cloud:P ). You pay for a memory and CPU multiplied. This means that when there's nothing to work on, you'll pay nothing (or almost nothing depending on the triggering mechanism). This is the moment when you pay for your application performing actions. If an app user can pay more than the cost of the execution, you'll be profitable. If not, maybe it's about time to rethink your business.

### Performance matters

With this approach and detailed enough measurements you can actually can see where you spend the most money. It's no longer profiling an app for seeing where is it slow or where it consumes most of the memory. It's about your business burning money in different places. Whether to update one or not - it's a decision based on money and how much does it cost to fix it. With highly profitable businesses you could even flood your less performing parts with money. Just like that.

### Environments and versioning

How to version a function? Preserve signature and rewrite it. Then deploy. Nothing less nothing more. I can almost see a new wave of development approaches where Continuous Delivery looks like a grandpa trying to run with Usain Bolt. You can't compete with this. It's a brand new league.

### Summary

If you think about areas you should invest your time, public cloud and functions are the way to go. For majority of the cases, this is going to be vital to survive in the market competing and betting for the lowest costs of infrastructure, IT and devops.
