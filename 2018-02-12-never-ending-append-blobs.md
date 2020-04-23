---
layout: post
title: "Never ending Append Blobs"
date: 2018-02-12 09:55
author: scooletz
permalink: /2018/02/12/never-ending-append-blobs/
categories: ["Azure", "design"]
tags: ["Azure", "design"]
nocomments: true
---

In this article I'll describe an easy and fast way to use Azure Storage Append Blobs to create a never ending Append Blob. Yes, a regular Append Blob has its limitations, including the maximum number of blocks and the size, but with a proper design we can overcome them.

### Limits we want to overcome

According to [Azure Subscription Storage Limits](https://docs.microsoft.com/en-us/azure/azure-subscription-service-limits#azure-blob-storage-limits), an Append Blob is limited in the following way:

1. Max number of blocks in an append blob: 50,000 - this means that we can append to a single blob only 50,000 times, no matter how much data we add at the same time
1. Max size of a block in an append blob: 4 MiB - this means that a single operation adding one chunk of data, cannot contain more than 4 MiB.

If we could address the first, it's highly unlikely that we'd need to address the second. 4 MiB is just enough for a single append operation.

### Overcoming the limited number of blocks

Let's consider a single writer case. Now, agree that we'll use natural number (1, 2, 3, ...) to name blobs. Then, whenever an append operation is about to happen, the writer could check the number of already appended blocks by fetching blob's properties and create another one, if the number is equal to the max number of blocks. We could also try to append and catch the `StorageException`, checking for `BlockCountExceedsLimit` error code (see [Blob Error Codes](http://BlockCountExceedsLimit) for more). Then, we'd follow with creating another blob and appending to the newly created one. This case is easy. What about multiple processes, writers trying to append at the same time?

### Multiple writers

Multiple writers could use a similar approach. There's also a risk of not being able to check for the limit. When you fetch attributes, another writer could already append their block making the number invalid. We could stick with the exception handling way of doing it:

1. get the latest blob name (1, 2, 3, ... - natural numbers)
1. append the block
1. if (2) this throws:
    1. try to create the next one or retrieve existing one
    1. append the block

This allows multiple writers to write to the logically same chunked blob, that is split across multiple physical Append Blobs. Wait a minute, what about ordering?

### Ordering multiple writers

With multiple writers A, B, C, ... appending blocks

1. A1, A2 - for A,
1. B1, B2 - for B,
1. C1, C3 - for C3

the following sequences could be a result of applying this approach:

1. A1, A2, B1, B2, C1, C2
1. A1, B1, B2, A2, C1, C2,
1. A1, B1, B2, C1, C2, A2

You can see that this creates a partial order (A1 will always be before A2, B1 before B2, C1 before C2) but different total orders are possible, depending on the speed of writers. Usually, it's just ok as the writers were appending to a blob their results, their operations, not carrying about the others' results.

### Summary

We've seen how easy it's to implement a never ending append blob for multiple writers. This is a great enabler in case, where you need a single logical, log-like, blob, that provides an ordered list of blocks.
