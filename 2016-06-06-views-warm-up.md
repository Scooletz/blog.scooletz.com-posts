---
layout: post
title: "Views' warm up for Event Sourcing"
date: 2016-06-06 09:00
author: scooletz
permalink: /2016/06/06/views-warm-up/
nocomments: true
categories: ["Design", "Event sourcing"]
tags: ["event sourcing", "performance", "views"]
imported: true
---

When using Event Sourcing as a foundation for your solution, the command part is a solved problem. Just take an aggregate version, a command, apply onto a state and try append created events to a store, checking the version again. There is a read part of this as well, called views, which is nothing more than an aggregation of a subset of events from the system. This works like a live query, which consumes events from a log and applies them on the projection on and on. Considering that the number of events is constantly growing, how would you deploy a new version of application containing a new view which needs to be build from the beginning, from the very first event? Even with a well performing database applying a few millions of events can take a while.

### Warm up routine

Let's consider a following routine. Instead of calling views by their names, a system version is appended. Take a view 'users' as an example

1. for version 1.0.0 it's "users-1_0_0"
1. for version 1.2.0 it's "users-1_2_0"

Before publishing the new version and moving all users to use it, predeploy the application and run its view builder. The views will be rebuild in the background, taking needed time. Once the builder starts to have problems with getting last events as there's no more data, the views are prebuilt and a new version can be deployed.

### Cost and optimization

Of course these views rebuild can be tedious and long. These could increase costs of your app as well as put an additional pressure on the event store. If a cost/performance optimization is needed, you can consider detection of a view change, something very similar to [what Rinat did](https://abdullin.com/post/getting-rid-of-cqrs-view-rebuilds/) a few years back. You may come up with something more explicit as well. For any mechanism, the rule would be that if a view is the same, your app uses the last existing version of the view.

Providing this warm up routine, especially when using blue green deployments can improve not only the performance of your application start (the new version scenario) but also can provide an environment for testing the deployment before switching to the new version.
