---
layout: post
title: "Azure Functions: processing 2 billions items per day (1)"
date: 2017-11-30 09:55
author: scooletz
permalink: /2017/11/30/azure-functions-processing-2-billions-items-per-day-1/
nocomments: true
image: /img/2017/11/serverless1.jpg
categories: ["Azure", "Azure"]
tags: ["2billions", "azure", "FaaS", "functions"]
imported: true
---

In this series I'll describe a few patterns that enabled me to process 2 billions items per day using Azure Functions. Yes 2 billions items per day. The aim of this trial was not to check whether you can do it with Azure Functions. You can do it easily. The goal was to do it in a cost-aware and cost-wise manner, enabling fast processing with a small amount of money spent on this.

### Initial phase

The start point was simple. To have a single queue, in my case Azure Storage Queue, and simply enqueue items to it, and run processing on a Consumption Plan. This looked pretty nicely. If you ever try Azure Functions you'll see the ability to scale up instances when needed, just to make your workload processed in a timely manner.

I must admit that I skipped that part. When you calculate the number of operations that a single queue can handle, it won't be enough to cope with 2 billions item per day. Yes, you can scale to multiple queues or use a different kind of queue. This was not the case for my experiment though.

### It comes in batches

The important part that I intentionally didn't mention, was the fact, that the numbers of items' producers was limited. Also, they were able to batch items and flush them once in a while. With this assumption I was able to use a dense serialization protocol (big no no for JSON) and fill every single message that is being sent with hundreds, sometimes, thousands of items to get them processed.

In my case this lowered the number of messages greatly, by a factor of 1000, leaving the whole thing working as it was supposed to. Yes, the receiving part become a bit different as it was required to deserialize the densely packed payload properly.

You may ask why not [Event Hubs](https://azure.microsoft.com/en-us/pricing/details/event-hubs/)? Being able to pack data on my own, being given the possibility of a delayed write and comparing prices for the scale I talk about, Azure Storage Queues with a properly selected serializer still won in my calculations.

### <del>Cheating</del> Seeing opportunities

This was the first opportunity that I used to make the processing faster and cheaper. We saw that using a batch (smart-batching in this case) greatly lowered the number of moving pieces, still delivering the same value. In the following entry, we'll move a bit deeper into the solution I built.
