---
layout: post
title: "Beyond the Cache Line: Data Access Alignment for Predictable Storage Performance"
date: 2025-10-29 06:00
author: scooletz
permalink: /2025/10/29/beyond-the-cache-line
image: /img/2025/beyond-the-cache-line.webp
categories: ["performance", "dotnet"]
tags: ["performance", "dotnet"]
whitebackgroundimage: true
twitter: false
---

# Beyond the Cache Line: Data Access Alignment for Predictable Storage Performance

If you’ve ever attended a performance conference talk or read a performance related blog post, you might have heard a mighty word that is used much too often: the **alignment**. The word itself has a very simple meaning: to put something in line, according to provided boundaries. Whether it’s a CPU, storage access or fetching a page of memory, modern computing is filled with boundaries that you need to align to.

# Data Misalignment Costs

The mentioned boundaries are all over the place. We could start with a CPU, the core chip of your computer. One of the boundaries that is often mentioned is a CPU cache line that usually occupies 64 or 128 bytes. As CPU caches are loaded and offloaded in lines, so if you design your data structures so that they keep their data together, it can drastically improve the performance. This happens naturally for arrays of struct (value types). If you have an array of integers and you scan it in a loop, it will be really fast due to the data collocation.

There’s also another side to this coin. If you have multiple threads touching the same cache line, and writing data to it, you’ll pay the penalty of *false sharing*. The performance of multithreaded write operations will be impacted by the constant switching of the ownership of the given cache line.

We can see that aligning to the boundaries given by a piece of computing that we interact with is important on the CPU level. Is it the CPU only? Maybe there are other aspects of modern computing that we should consider?

# The Size Doesn’t Matter

The general rule of the aligned access can be applied further to the storage layer as well. The storage, whether it’s your local SSD or a remote cloud storage like S3/Azure Blobs, will likely operate in blocks. It might be hidden from the end user, but the underlying truth is that it blocks all the way down. If you read one byte or ten bytes, the access will be block based. The same with writing operations. How can it be if we can *pwrite* (write to a file) one byte, right? What is the block?

For your local storage, if you access your disk with a low level file API, it will require you to align your data to the *sector size* that you can obtain for each disk. If you use memory mapped files, the alignment will be equal to the page size of the system. For instance if your system page size is 4KB, even if you read just one byte from a single page, it will be loaded as a whole to the memory.

You can see it again, that you need to pack your data nicely. Then, the more predictable the walk across data, the better. You should read from and write to the same page multiple times and only then move to the next. Unless you want to be slowed down due to frequent page loads/unloads.

# Heavy Cloud

The access penalty will be even more visible if you use a public cloud provider. The first disadvantage is that it takes a lot of time to access remote storage in the cloud. It’s not microseconds, we’re talking tens of milliseconds here. Multiply it by the number of operations and you get seconds, minutes, (I hope not) hours.

This can get worse. The unpredictably often accessing the storage, can effectively apply a denial of wallet attack on your application. In the cloud you pay for every access to the storage services (S3, Azure Storage). The lower the number of requests you send, the cheaper your application will be.

Not only you can benefit from the speed, but also from not burning a lot of money.

# Batch as An Opportunity

Batching a few operations together is always a good way to amortize the cost of performing them. Imagine that you perform N writes to a file and then you call a single *FSYNC*. You just amortized the heavy cost of FSYNC across all of them. This is literally what every single database (not in-memory, serious one, not dropping data) will do. Or you write data to a cloud storage. You bulk them in a memory and then send them in one go. Again, network costs \+ operation feeds amortized.

There’s one more opportunity hiding here though. What you can do is prepare data to make these costly operations… cost less. One thing is compression. Usually databases are IO bound, sending less data to the storage and requesting less can be beneficial. But this is a trivial thing. You just compress before writing and decompress after reading them.

Let’s now assume that you use a memory mapped access to data. Let’s recall, a page will usually be of size 4KB. Assuming that you need to touch a lot of them, highly likely performing a lot of loads and unloads, would it matter to make it in some specific order?

# Sort the Batch

What if we sort the batch of operations before performing them, so that they are aligned to the way they reside on disk? The more aligned ordering, the less likely loading/unloading for pages. Ultimately, we should look towards loading and unloading each page only once, right? If we sort it in a way the data are stored, this gets as predictable as it can be. Where can we apply such optimizations then?

One of the areas where such optimizations shine are databases. A database will read and write lots of data. You could even claim that any database is storage bound, meaning that the storage access is the most likely to be the bottleneck in every single profiling session you perform. Going further, a particular part of databases that requires a lot of bookkeeping and updating data in random places is indexing. The properties that you index can change arbitrarily causing different parts of the storage to be updated. Solving this in an effective way is not a trivial task to handle. This is why [RavenDB](https://ravendb.net/?utm_source=blog.scooletz&utm_medium=link&utm_campaign=blogposts)
 built their own indexing engine called [Corax](https://docs.ravendb.net/7.1/indexes/search-engine/corax/?utm_source=blog.scooletz&utm_medium=link&utm_campaign=blogposts). As Corax work focuses heavily on making your data searchable, the faster and more efficient it works, the faster your data is queryable. Also, please notice that the less storage throughput it uses, the more is left for other tasks (again: the storage as the main bottleneck). Having this in mind, let’s take a look into the following PRs.

In the [PR](https://github.com/ravendb/ravendb/pull/20980) *Optimizing Index Writes*, ordering is based on how the given field will be stored. As we traverse through the ordered data, we can push the performance even further, requesting pages to be prefetched before we access them. It gets as sequential and as predictable as possible. Fewer jumps, accessing one after another.

The other [PR](https://github.com/ravendb/ravendb/pull/21325) *Streamlining Posting List and Lookup Tree Processing* performs a slightly different, but related, twist on data. Instead of zipping two processes A and B, where it alternates between the two sequences of pages that it needs to traverse, it goes through A first, then through B. Again, this reduces a potential need for loading/offloading pages, and keeps the application of the data colocated. 

# Takeaways

RavenDB benefits greatly from such optimizations. After all, it wants to be as fast as possible. Your applications can also benefit from similar data processing techniques:

1. **Design Data Structures for Storage**: Thinking about physical layout when designing logical schemas. What happens on writing data , what happens when reading data.  
2. **Consider the hot path**: optimize for the scenario that is in the hot path.  
3. **Be Cloud Aware**: Understand the I/O characteristics and pricing models of different cloud storage services (e.g., S3, Azure Blob Storage)

May the data and your accessing patterns align in your favor\!
