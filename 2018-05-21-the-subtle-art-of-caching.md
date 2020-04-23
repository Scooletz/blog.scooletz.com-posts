---
layout: post
title: "The subtle art of caching"
date: 2018-05-21 08:55
author: scooletz
permalink: /2018/05/21/the-subtle-art-of-caching/
image: /img/2018/05/cache.png
categories: ["architecture", "cache", "design"]
tags: ["architecture", "cache", "design"]
whitebackgroundimage: true
---

### TL;DR

It's often said that *there are only two hard things in Computer Science: cache invalidation and naming things*. In this blog post we'll reconsider this statement against the modern databases and approaches. We'll start with no-cache rule and move to some situations where applying caching might be good, or even, vital.

If you never used a cache personally (the operating system did it for you), please start from the beginning. If you're a caching pro, just skip a few paragraphs.

### What is caching anyway

Caching is a simple mechanism, a component, that stores some of the data returned from a medium, for future requests. This enables responding to requests much faster, returning a possibly stale answer. There's no free lunch in caching. If you cache, you must accept that sometimes the response will be stale.

### The first rule of caching is you do not cache

I've seen a lot of applications that worked without any caching at all. If you're in the managed world, like .NET, and you just focus on proactively solving some of the most common performance issues (allocations mostly), you can easily do without any caching. Although I've seen a lot of **applications** without caching, this is not the case when talking about **systems**.

### Systems and caching

First, let me define how I understand a system

> A **system** is a set of applications, services, connected to deliver a set of business functionalities.

This means that whenever your service calls a payment gateway, an email service or the currency rate service, I'll consider them as a system. Now let's take a look at the currency rate sample. Let's imagine that your app first calls the currency rate service, and then, issues a payment.

```csharp
var rate = currencyService.GetRate("Euro", "$");
var amoutOfDolars = amountOfEuro * rate;
paymetService.Pay (amoutOfDolars);
```

As you can see in the sample above, first we called the currency service and obtained the data. Secondly, we called the other service with the data calculated on the basis of the first call. Now, what happens if between these lines the exchange rate changes? We didn't do any caching but still, we can use stale data. This isn't an explicit caching case, but still, you should consider it when **designing systems**.

### Caching immutable is easy

We know that we might "cache" things implicitly. But let's focus now on caching things explicitly. What would be the easiest thing to cache, without worrying about data becoming stale? Of course, some immutable data. What could it be? Let me give you a few examples:

1. a single git commit cached under it's SHA1 signature - a single git commit cannot be altered. If you altered it, it would have a different SHA1 signature so it would be a different commit
1. a historical data, for example a currency table from the last Monday - it happened, nothing can be changed in the past
1. a version of something - your document in version 5, will never ever be in version 5 again. You can cache it forever

> Caching immutable is stale-safe. Even if different parties have different versions of the cached item, they know explicitly the version or the date of the cache entry and might act accordingly.

### Snapshots, mementos and log-based approaches

There are cases where it's better to capture all the changes applied to a single entity, rather than saving the whole entity every single time. You've probably heard of **event sourcing** or other log-based approaches. Instead of saving the state of an entity, we could save all the changes that were applied to it. A simple example could be a calculator with all the operations:

1. Added (1)
1. MultipliedBy (3)
1. StoredInMemory()

Every entry/event, has a position in this ordered list. Also, applying all the entries from the initial state (0 on our calculator) will provide always the same result (pressing the same buttons in the same order). There's an optimization that we could do if the number of entries is tooooo big to process every time we access our calculator (imagine pressing 1000 keys again and again). We could store the value on a piece of paper and write the position of the entry that we applied. Effectively, we just store **a version of the calculator state** which can't become stale. This is a **snapshot**, or a **memento**. Once we want to receive the latest version, you can get the recent snapshot/memento, read the version it was done for and read all the rest (the tail) of entries that were stored after the version when snapshot was taken. For small entities, it won't help much. If we discuss a document, that went through hundreds of small editorial changes, this approach might save a lot of bandwidth.

### What if-not-so-much-modified

The last but not least is strongly related to **ETags**, **Last-Modified** and caching in the web (http protocol). If you had an entity that isn't frequently changed, you could think about storing it in a cache with its version. Whenever a party asks for the item, you could query the underlying store or a database, passing the previously stored version. If the value didn't change, it would just return a status. If it changed, the new value would be returned with a new version. You could think of this query as the following statement in SQL

```sql
SELECT  data, version FROM MyTable
WHERE id = @id AND version <> @version
```

If the version is the same, nothing is returned. If the version is different, this query will return a single row (with the specified ID) and the recent version.

> In the http protocol, **ETag** is nothing more or less but a version of a specific resource.

The same approach can be used in some of the **cloud databases** that provide http based API. Quite often, they respect http headers, so for data that are frequently used and infrequently changed, you can use ETags to query only for the data in case they changed. This approach, by lowering the number of bytes transmitted, can decrease the cost and increase the throughput (not sending same data over and over again).

### Going wild

The last option, that I truly don't like, is *just enable caching(TM)*. If you apply caching blindly, like any other tool, you might get hit by its usage quite fast. Reading **stale data**, not reading your writes, processing requests as something is set but it is not. All these phenomena are just before you if you *just enable caching(TM)*. Think and think again. And before caching everything, go through the previous points looking for opportunities to use caching in a right way.

### Summary

Think about caching as one of the tools that you can use. Don't apply it to everything. Search for big entities. For small entities this might not make a change. The bigger the entities are, the bigger performance gains you will notice by not transporting the payload over and over again. Also, design for cacheability. It's not something you get for free.
