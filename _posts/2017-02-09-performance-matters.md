---
layout: post
title: "Performance matters"
date: 2017-02-09 09:55
author: scooletz
permalink: /2017/02/09/performance-matters/
nocomments: true
image: /img/2017/01/stocksnap_uzao7l631q.jpg
categories: ["Optimization"]
tags: ["Marten", "performance"]
imported: true
---

### TL;DR

This is a short follow-up post about Marten's performance. It shows that saved allocations are not only about allocations and memory. It's also about you CPU ticks, hence the speed of your library.

### Moaaaaar performance !

Let me present you three pictures comparing performance before and after removing a lot of allocations. They were provided by [Jeremy](https://jeremydmiller.com) after benchmarking my PRs to Marten. My work was purely focused on allocations, but additionally, as shown below, it improved Marten's speed of execution.

### **Events**

The speed improvement isn't that significant, but please take a look at the allocated bytes. Now it takes much less memory required before

![events](/img/2017/02/events.png)

### Documents

The new insert is 10% faster and takes much less memory than before.

![docs](/img/2017/02/docs.png)

### Bulk inserts

Here, after [enabling npgsql library to accept ArraySegment<char>](https://github.com/npgsql/npgsql/pull/1411) I was able to reuse the same pooled writer. The new approach not only skips allocations but also leases a pooled writer only once. Just take a look at these numbers!

![bulk_loading](/img/2017/02/bulk_loading.png)

### Summary

When working on a library or a tool it's good to think about performance and memory consumption. Even in a managed Garbage Collected world, using pooling for buffers or objects at all might not only reduce a memory consumption but also improve the overall speed of your creation.
