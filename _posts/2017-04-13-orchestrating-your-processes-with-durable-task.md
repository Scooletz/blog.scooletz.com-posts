---
layout: post
title: "Orchestrating processes with full recoverability"
date: 2017-04-13 08:55
author: scooletz
permalink: /2017/04/13/orchestrating-your-processes-with-durable-task/
image: /img/2017/04/stocksnap_rjwnf1gvhz.jpg
categories: ["Azure", "Service Fabric"]
tags: ["async", "azure"]
imported: true
---

### TL;DR

Do you call a few services in a row as a part of a bigger process? What if one of the calls fails? What if your hosting application fails? Do you provide a reliable way for successfully finishing your process? If not, I might have a solution for you.

### Anatomy of a process

A process can be defined as at least two calls to different services. When using a client library of some sort and C# *async-await* feature one could write a following process

```csharp

var id = await invoiceService.IssueInvoice(invoiceData);
await notificationService.NotifyAboutInvoice(id);

```

It's easy and straightforward. First, we want to issue an invoice. Once it's done, a notification should be sent. Both calls although they are *async* should be executed step by step. Now. What if the process is halted after issuing the invoice? When we rerun it, there's no notion of something stopped in the middle. One could hope for good logging, but what if this fails as well.

### Store and forward

Here comes the solution provided by [DurableTask library](https://github.com/Azure/durabletask) provided by the Azure team. The library provides a capability of recording all the responses and replaying them without execution. All you need is to create proxies to the services using a special orchestration context.

With a process like the above when executing following state is captured:

1. Initial params to the instance of the process
1. *invoiceData* are stored when first call is done

1. *invoiceService* returns and the response is recorded as well

1. *invoiceNumber* is stored as a parameter to the second call

1. *notificationService* returns and it's marked in the state as well

As you can see, every execution is stored and is followed by storing it's result. OK. But what does it mean if my process fails?

### When failure occurs

What happens when failure occurs. Let's consider some of the possibilities.

If an error occurs between 1 and 2, process can be restarted with the same parameters. Nothing really happened.

If an error occurs between 2 and 3, process is restarted. The parameters to the call were stored but there's no notion of the call to the first service. It's called again (yes, the delivery guarantee is *at-least-once*).

If an error occurs between 3 and 4, process is restarted. The response to the call to the invoice service is restored from the state (there's no real call made). The parameters are established on the basis of previous values.

And so on and so forth.

### Deterministic process

Because the whole process is based either on the input data or already received calls' results it's fully deterministic. It can be safely replayed when needed. What are not deterministic calls that you might need? *DateTime.Now* comes immediately to one's mind. You can address it by using deterministic time provided by the [context.CurrentUtcDateTime](https://github.com/Azure/durabletask/blob/master/Framework/OrchestrationContext.cs#L31).

### What's next

You can build a truly powerful and reliable processes on top of it. Currently, implementation that is provides is based on Azure Storage and Azure Service Bus. In a branch you can find an implementation for [Service Fabric](https://github.com/Azure/durabletask/tree/vnext_servicefabric), which enables you to use it in your cluster run on your development machine, on premises or in the cloud.

### Summary

Ensuring that a process can be run till a successful end isn't an easy task. It's good to see a library that uses a well known and stable language construct of async-await and lifts it to the next level, making it an important tool for writing resilient orchestrations.
