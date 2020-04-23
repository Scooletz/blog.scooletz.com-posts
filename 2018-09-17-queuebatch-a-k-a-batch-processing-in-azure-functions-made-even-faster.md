---
layout: post
title: "QueueBatch a.k.a. batch processing in Azure Functions made even faster"
date: 2018-09-17 08:55
author: scooletz
permalink: /2018/09/17/queuebatch-a-k-a-batch-processing-in-azure-functions-made-even-faster/
nocomments: true
image: /img/2018/09/batch.png
categories: ["Azure", "serverless"]
tags: ["Azure", "serverless"]
whitebackgroundimage: true
---

[QueueBatch](https://github.com/Scooletz/QueueBatch) is a project that I recently opened. It allows an efficient processing of Azure Storage Queues' messages, triggering your function not for every single item, for batches.

### Why batching

Why batching? I wrote some time ago about batching, specifically [smart batching](http://blog.scooletz.com/2018/01/22/the-batch-is-dead-long-live-the-smart-batch/) approach. Living in the era of cloud means that you can easily scale out when needed and if needed. If, because of the batching multiple operations into one, one can issue less IO transactions, access less resources and, eventually, pay less, I'm for it.

### Web jobs 3.0-ish

If you're familiar with Azure Functions, you probably know [the issue](https://github.com/Azure/app-service-announcements/issues/129) that describes all the breaking changes that are being introduced (when I write it) to the WebJobs SDK. They changed a few public APIs, which resulted in some changes in QueueBatch. After all, it's a regular trigger with a listener that is based on SDK APIs. Hopefully, when the stable 3.0 is released, I'll be able to release a stable version of QueueBatch.

### Storage SDK is not so fast

One of the improvements that I provided is a custom parser for responses of *GetMessages* requests. I did it in [a single PR](https://github.com/Scooletz/QueueBatch/pull/2) so you can follow the changes if you want. I know that SDK client provides a lot of options for retrying, calling the secondary endpoint etc. Unfortunately, for parsing responses, it uses *XmlReader* built on top of the response stream. This isn't either fast or efficient. Knowing the schema of the response, I created a custom parser that you can opt-in for, to get more performance. Below, you can find the benchmark measuring a single parsing operation.

![QueueBatchFaster](/img/2018/09/queuebatchfaster.png)

It's important, that the *QueueBatch* provides the payload of a message as a *Memory<byte>*. This allowed using a single *byte[]* for the whole request, copy it from the response stream and provide chunks of it in a lightweight *Message* structs. This means, no allocations for the payload. The numbers above speak for themselves.

### Stability

QueueBatch started as a component of something bigger. My idea behind open sourcing it was simple - make it accessible for other people thinking about money they pay for Azure Functions and wanting something much faster/better when operating on the old school Azure Storage Queues. A stable version of QueueBatch will be released as soon as WebJobs SDK is released as stable 3.0 and I have it running for some time for my purposes.

### Summing up

The batch is dead and Azure Storage Queues are one of the oldest services provided by Azure. Still, if we use the smart batch approach with ASQ as underlying transport, the throughput and latency you can squeeze out of it is amazing.
