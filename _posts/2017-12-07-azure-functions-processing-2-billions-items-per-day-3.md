---
layout: post
title: "Azure Functions: processing 2 billions items per day (3)"
date: 2017-12-07 09:55
author: scooletz
permalink: /2017/12/07/azure-functions-processing-2-billions-items-per-day-3/
nocomments: true
image: /img/2017/12/serverless.png
categories: ["Azure"]
tags: ["2billions", "azure", "FaaS", "functions"]
imported: true
---

Here comes the third entry in a series in which I'm describing a few patterns that enabled me to process 2 billions items per day using Azure Functions. The goal was to do it in a cost-aware and cost-wise manner, enabling fast processing with a small amount of money spent on this.

1. [part 1](http://blog.scooletz.com/2017/11/30/azure-functions-processing-2-billions-items-per-day-1)
1. [part 2](http://blog.scooletz.com/2017/12/04/azure-functions-processing-2-billions-items-per-day-2/)
1. part 3

The first part was all about batching on the sender side. The second part, was all about batching on the receiver side. In this part we'll move to truly backendless processing.

### No backend no cry

I truly admire how solutions are migrated to the serverless world. The most interesting is observing 1-1 parity between components that were there before and functions that are created now, a.k.a "Just make it a func!". If you see this, one to one mapping, there's a chance that you're migrating code without changing the approach at all. Let me give you an example.

Imagine that you need to accept users' requests. These requests are extremely unlikely to fail (there are ways to model services towards that) and if they do, there's a natural compensation action. You could think that using a queue to store them is a perfect way of accepting a request, that can be processed later on. OK, but we need a component that will accept these requests. We need something that will write to one of Azure Storage Queues, right? Wrong.

### Tokenzzzzz

Fortunately for FaaS, Azure Storage Queues have a very interesting capability. They can be accessed directly with a limited scope of rights. This functionality is provided with [SAS tokens](https://docs.microsoft.com/en-us/azure/storage/common/storage-dotnet-shared-access-signature-part-1) that enable access to Add, Update and/or Process, and more. What you can do is to give somebody access to only Add messages, you can limit this access to 5 minutes (and revalidate if user can do it after this period of time). The options are limitless.

If we can limit the access to a queue to just adding messages, why would we need a function to accept it? Yes, we might need a function to issue a few tokens at the beginning but there's no need of consuming a regular request and move it to a queue. No need at all. Your user can use a storage service directly with no code for putting data in there.

To put it even more bluntly: You don' need a user to call a func to put a message in a queue. A user can can just put a message.

### Cloud native

This moves us to being cloud native. To embrace fully different services and understand that using them no longer requires writing code for them. Your functions can easily move to a higher level, assigning permissions, returning tokens and shifting from being a regular app that "just was migrated to functions" to a set of "cloud native functions", from "using services" to "orchestrating their usage".

### Where's the cherry

We've got the cake. We need a cherry. In the last part, I'll briefly describe costs and numbers. See you soon.
