---
layout: post
title: "Azure Functions: processing 2 billions items per day (2)"
date: 2017-12-04 09:55
author: scooletz
permalink: /2017/12/04/azure-functions-processing-2-billions-items-per-day-2/
nocomments: true
image: /img/2017/12/serverless1.jpg
categories: ["Azure"]
tags: ["2billions", "azure", "FaaS", "functions"]
imported: true
---

This is the second blog post in a series in which I'm describing a few patterns that enabled me to process 2 billions items per day using Azure Functions. The goal was to do it in a cost-aware and cost-wise manner, enabling fast processing with a small amount of money spent on this.

1. [part 1](http://blog.scooletz.com/2017/11/30/azure-functions-processing-2-billions-items-per-day-1)
1. part 2

In the first part you saw that batching can greatly lower the number of messages you need to send, and that it can actually broaden a selection of tools you can use to deliver the same value. My choice was to stick to good, old fashioned Azure Storage Queues as with the new estimated number of messages, I could simply use a single queue.

### Serverless side

The initial code responsible for dispatching messages was simple. It was a single function using [*QueueTrigger*](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-queue), dispatching messages as fast as they go. Because of running under Consumption Plan, all the scaling was being done automatically. I could see a flood of log entries informing about functions being properly executed.

The test was run for a week. I checked the amount of money being spent in the new Cost Management tool and refactored the code a little bit. I was paying too much for doing lookup after lookup and spending too much time on finding data needed for the message processing. The new version was a bit faster and a bit cheaper. But it made me think.

If a single Table Storage operation takes ~30-40 ms, and I need to do a few for a single function run, what am I paying for? Also, I knew that the data are coupled temporarily. In other words, if one entry from a table was used for this message, it's highly likely to be used within few seconds. Also, I did not care about latency. There was already a queue in there in front of it. I was fine whether the result will be presented within 1s or 5s. I asked myself: how can I use all these constraints in my favor?

### Processing batches in batches

The result of my searches was as simple as that. Why don't process messages already containing batched entries in batches as well. I could use a [*TimerTrigger*](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer) to get this function run every 5/10 s and grasp all the messages using a batched operation *GetMessages* from Azure Storage Queues. Once, they are fetched, I could be able to either prefetch all the required data using parallel async operations with *Task.WhenAll* or use a local cache for the execution.

Any side effects of dispatching messages on my own? Good poison message handling and doing some work that was internally handled by *QueueTrigger*.

The outcome? A single function running every x seconds, draining the queue till it's empty and dispatching loads of messages.

Was it worth it? The total time spent previously by functions could have been estimated as

> total_time = number_of_messages * single_message_processing_time

where *single_message_processing_time* would include all the lookups.

With the updated approach, the number of executions was stable (~15k per day) with different processing times, depending on the number of messages in the queue. The most important factor was the amortized cost of lookups and storage operations. The final answer was: yes, it was definitely worth it as it lowered the price greatly.

### Moving on

In this part we saw that the batching idea leaked to the serverless side, beautifully lowering the time and the money spent on the function execution. In the next part we'll see the power of backendless.
