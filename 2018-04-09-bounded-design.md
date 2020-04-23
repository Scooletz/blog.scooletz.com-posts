---
layout: post
title: "Bounded design"
date: 2018-04-09 08:55
author: scooletz
permalink: /2018/04/09/bounded-design/
categories: ["architecture", "design"]
tags: ["architecture", "design"]
nocomments: true
---

If you wake up a Domain Driven Design fan and scream the word "Bounded" the answer will be immediate and always the same: "Context". It's funny that this word is having so much trouble in leaving DDD context. I'd like to encourage you for broadening it a little bit, to a design, architecture space.

### Reality strikes back

It's quite interesting that often, when we hit a limitation of a service of any kind, the first reaction is negative. Probably you heard statements similar to these:

* *Why did they set the value at this level?*
* *I want to be able to put a transaction on top of anything*
* *I don't need to change my design. It's their fault that the created a service that sucks*

I'm not writing about malicious providers claiming, that they do something when they don't. I'm talking about a well described limitations that you can find in a documentation of a tool that you're using. You can try to kick the wall, but still, the limitation will be  there. The better question to ask is why?

### Limit to profit

Every limitation that you put in your service provides more space for designing it. Let's take into consideration a supported type of data for a custom database. If all values and keys could be only of type int (4 bytes), then I would not need to care about variable lengths of buffers used for storing them! Every pair of a key and a value would be written on 8 bytes! And this is memory aligned as well! And this and that. Let's see another example.

Often, when using cloud databases there's a notion of a partition or a shard. You are promised to have some kind of transactionality within one partition, but you can't use one transaction across partitions. Again, you may raise all the blaming questions, or think about partition as a unit of a scalability. Imagine that you'd have to store all the data of a single database on one machine. That's at least highly inefficient. Now think. You create partitions from the very beginning. They can be moved to other machines, as all the data of one partition resides on one machine (this is implication, so multiple partitions can reside on the same machine). This could be a game changer, especially, if you're writing a cloud service like Azure Storage Tables.

### Summary

You can see the pattern now. Behind majority of these limitations, are some design decisions. They were made to make a service work. Of course there are probably some sloppy ones, but still, whenever you see a limitation like this, think about it. Then, design your solution accordingly.
