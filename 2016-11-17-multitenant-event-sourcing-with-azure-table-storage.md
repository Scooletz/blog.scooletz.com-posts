---
layout: post
title: "Multitenant Event Sourcing with Azure Table Storage"
date: 2016-11-17 09:50
author: scooletz
permalink: /2016/11/17/multitenant-event-sourcing-with-azure-table-storage/
image: /img/2016/11/stocksnap_emlmui5x69.jpg
categories: ["DDD", "Event sourcing"]
tags: ["azure table storage", "design", "event sourcing"]
imported: true
---

### TL;DR

Designing a multitenant system puts a hard requirement on a designer to do not leak data between tenants. Is there anyone who would like to show a list of employees' emails from one company to another?

### **Azure Table Storage**

Azure Table Storage is a part of Azure Storage Services. It's mentioned in the original Windows Azure Storage whitepaper and provides a stable foundation with known limitations, quotas and API that hasn't changes for ages (ok, years). The most important aspect of it is the throughput which limited in two dimensions:

* partitions - a partition is defined by a partition key value and can CRUD at most 500 entities (rows) per second
* storage account - an account can process at most 10k operations per second

These two numbers can impact the performance of an app and should be taken into consideration when designing storage.

### **StreamStone**

There's a library which provides an event sourcing store on top of the Azure Table Storage. It's called [StreamStone](https://github.com/yevhen/Streamstone). It provides a lot of capabilities but not a *from-all* projection (see this [PR](https://github.com/yevhen/Streamstone/issues/24) for more info, including my notes). This can be added (not easily), which I've done introducing some overhead on the write side.

Having a storage problem solved, how would you define and design a multitenant system?

### **One to rule them all**

The initial attempt could be to add the company identifier to the partition key. Just use it as a prefix. That could work. Until one of the following happens:

* a scan query without a condition is issued - just like SELECT * without where, yeah, that would be scary
* a company uses our app in a way that impacts others - it's easy, you need to saturate 10k operations per second

It looks that this could work, but it could fail as well. So it's not an option.

### **Separated accounts**

Fortunately, Azure provides [management API](https://msdn.microsoft.com/en-us/library/azure/ee460790.aspx) for storage accounts. This means that from an application, one can instantiate storage accounts under the same subscription but totally separated from each other. Like in a container or something. This boxes performance limitations for a company into its own account. The problem of a potential leakage is also addressed by storing data of a company in a totally separated account.

As mentioned by Adrian in the comments, there's a limit of [200 storage accounts per single subscription](https://docs.microsoft.com/en-us/azure/azure-subscription-service-limits#storage-limits) which is a high number to reach. Once you do it, additional layer of subscription management should be applied.

![stocksnap_g9pflyymu1](/img/2016/11/stocksnap_g9pflyymu1.jpg)

### Who knows them all?

Of course there's a need of a governor. A module that will know all the accounts and that will manage them. This, is a limited surface of possible leakage, leaving a good separation for the rest of the application/system.
