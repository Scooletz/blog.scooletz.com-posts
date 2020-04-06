---
layout: post
title: "Azure Functions: processing 2 billions items per day (4)"
date: 2017-12-14 09:55
author: scooletz
permalink: /2017/12/14/azure-functions-processing-2-billions-items-per-day-4/
nocomments: true
image: /img/2017/12/serverless1.png
categories: ["Azure"]
tags: ["2billions", "azure", "FaaS", "functions"]
imported: true
---

Here comes the last but not least entry in a series, where I'm describing a few patterns that enabled me to process 2 billions items per day, using Azure Functions. The goal was to do it in a cost-aware and cost-wise manner, enabling fast processing with a small amount of money spent on this.

1. [part 1](http://blog.scooletz.com/2017/11/30/azure-functions-processing-2-billions-items-per-day-1)
1. [part 2](http://blog.scooletz.com/2017/12/04/azure-functions-processing-2-billions-items-per-day-2/)
1. [part 3](http://blog.scooletz.com/2017/12/07/azure-functions-processing-2-billions-items-per-day-3/)
1. part 4

The first part was all about batching on the sender side. The second part, was all about batching on the receiver side. The third provided the way, to use Azure services without paying for function execution. The last part is about costs and money.

### How much do I get for free?

When running under Consumption Plan, you get something for free. What you get is the following:

* 400k GB-s - GBs means running with 1GB of memory consumed for 1s
* 1 million executions

The first item is measured with 1 ms accuracy. Unfortunately for the runner,

> The minimum execution time and memory for a single function execution is 100 ms and 128 mb respectively.

This means that even if your function could be run under 100ms, you'd pay for it. Fortunately for me, using all the batching techniques from the past entries, that's not the case. I was able to run function for much longer, removing the taxation of a minimal run time.

Now the second measure. On average there's over 2 million seconds in a month. This means, that if your functions is executed with smaller frequency, that should be enough.

### How much did I pay?

Not much at all. Below, you can find a table from Cost Management. The table includes the writer used for synthetic tests, so the overall it should be much lower.

![price](/img/2017/12/price.png)

This would mean that I was able to process 60 billion of items per month, using my optimized approach, **for 3$**.

### Is it a free lunch?

Nope, it's not. There's no such a thing like free lunch. You'd need to add all the ingredients, like Azure Storage Account operations (queue, table, blobs) and a few more (CosmosDB, anyone?). Still, you must admit, that the price for the computation itself is unbelievebly low.

### Summary

In this series we saw, that by using a cloud native approaches like SAS tokens, and treating functions a bit differently (batch computation), we were able to run under a Consumption Plan and process loads of items. As always, entering a new environment and embracing its rules, brought a lot of goodness. Next time, when writing "just a functiona that will be processed a few millions times per month" we need to think and think again. We may pay much less, if our approach truly embrace the new backendless reality of Azure Functions.
