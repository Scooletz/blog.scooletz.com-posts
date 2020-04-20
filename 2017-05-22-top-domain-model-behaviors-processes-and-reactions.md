---
layout: post
title: "Top Domain Model: behaviors, processes and reactions"
date: 2017-05-22 08:55
author: scooletz
permalink: /2017/05/22/top-domain-model-behaviors-processes-and-reactions/
nocomments: true
image: /img/2017/04/top_domain_model.jpg
categories: ["DDD", "Event sourcing", "Top Domain Model"]
tags: ["DDD", "Event sourcing", "Top Domain Model"]
---

### TL;DR

After pivoting two nights straight [here](http://blog.scooletz.com/2017/05/08/top-domain-model-ive-been-pivoting-all-night-long) and [here](http://blog.scooletz.com/2017/05/15/top-domain-model-ive-been-pivoting-all-night-long-again), it's time to settle down and ask is it possible to stick just to aggregates to model your domain? Is it possible to make it any better with services or maybe another tool is needed?

### Actions and state

Aggregates define the boundaries of transactions. That's their primary target. They narrow down not only our thinking (in a positive sense), but also provide an ability for scaling out our model. If every aggregate sits in its own bucket, there's nothing easier to save it as a document, or a stream of events within one partition. Their boundaries won't be ever crossed. The last one is actually a wishful thinking. Who would like to transfer money without ever receiving them on the other side?

### Actions and reactions

Think in processes. One account is debited, the other is credited. The second does not have to happen in the same transaction. Moreover, the second, in its nature, is a reaction to the first account being debited. In a natural world reactions often happen at the same time. Modelling them in this way (transactions across all the aggregates) might be more realistic but for sure, is not more useful. The more elements in the transaction, the higher probability of failing... and so on and so forth.

### Aggregates + processes

The above is a description of a perfect couple. Aggregates and processes are two tools that you can use to model any domain. Actually, if you dared to say that you don't need processes, I'd say that you could be wasting your time modelling a really boring and simple domain with a DDD tooling. From the tooling perspective, whether you want to event-source everything all the way or use good old fashioned orm with a message bus, it's up to you. Just remember, that the true power in your model comes from combining aggregates and processes.

### Summary

Modelling aggregates that just work (TM) is impossible. Any, even moderately complex domain contains a lot of reactions and processes that need to be captured in your model to make it usable. Next time, you think about an aggregate reacting to something, ask yourself, what kind of process is this, how should I name it, how to model it?
