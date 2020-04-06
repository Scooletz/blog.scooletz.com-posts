---
layout: post
title: "Anomalies: Snapshot Isolation"
date: 2017-09-25 08:55
author: scooletz
permalink: /2017/09/25/anomalies-snapshot-isolation/
image: /img/2017/09/si.jpg
categories: []
tags: ["anomalies"]
imported: true
---

This post starts a short series about different anomalies you may run into using modern databases or other storage systems.

### Snapshot Isolation to the rescue

Imagine a system that frequently deals with database locks and transactions that run much too long because of the locks being taken. Imagine that someone applies a magic fix, simply changing the isolation level to the **snapshot isolation**. Finally, this app is working, throwing an exception from time to time. Owners are happy, until they find, that somehow users are able to write more data than are allowed to write. The investigation starts.

### What are you made of, Snapshot Isolation?

If you wonder what Snapshot Isolation means, the description is quite simple. Instead of having locks on rows and checking whether or not a row can be locked/updated,etc, every row is now versioned. To simplify, imagine that a date is added to every row when it's modified somehow. Now, whenever a transaction starts, it has a date assigned, that creates a boundary for seeing newer records. Consider the following example:

1. BEGIN TX1
1. BEGIN TX2
1. TX2: INSERT row1 INTO tableA
1. COMMIT TX2
1. TX1: SELECT * from tableA

the last statement won't return **row1** as it was committed after transaction **TX1** started. Simple and easy, right? You can read only rows that were committed before you. What can go wrong then?

### Write skew

Now imagine a blogging service, that allows **only 5 posts** per one user. Let's consider a situation when a user has two employees entering posts for him/her. Additionally, let's assume that there are already **4 posts**.

1. BEGIN TX1
1. BEGIN TX2
1. TX1: SELECT COUNT(*) FROM Posts *returns 4*
1. TX2: SELECT COUNT(*) FROM Posts *returns 4*
1. TX1: INSERT post5a INTO Posts
1. TX2: INSERT post5b INTO Posts
1. COMMIT TX1
1. COMMIT TX2

As you can see, both transactions read the same number of posts: 4 and were able to add one more. Unfortunately, for the owners of the portal, now, their users know, that by issuing multiple requests at the same time, they can do much much more without paying for additional entries.

This error is called the **write skew**.

### Mitigations

The first mitigation you might think about is a simple update on the number of posts already published. Once a conflicting write is found, the database engine will kill the transaction. Another one could be replacing a record with itself. This still, qualifies as a conflict, and again, will kill the transaction committed afterwards. Are there any other tools?

Yes they are, but they are not available in every database. There's a special isolation level called **Serializable Snapshot Isolation** (SSI) that is less than 10 years old. It's capable of automatically checking whether or not two transactions overlap in a way, that one could impact another. One of the databases that is capable of doing it, is [PostgreSQL](https://wiki.postgresql.org/wiki/Serializable). Another one is the open source Spanner clone, called **CockroachDB**. Interestingly it defaults to SSI as it's described in [here](https://github.com/cockroachdb/cockroach/blob/8f2387e929a2e8d8487aede44ee47b42f94c433d/README.md#how-it-works-in-a-nutshell).

### Wrapping up

As always, don't apply things automagically, especially, if you deal with isolation levels. If you select one, learn how does it work and what anomalies are possible. When thinking about Snapshot Isolation, consider databases that support you with Serializable Snapshot Isolation, which removes the burden of updating rows "just-in-case" and can actually prove correctness of your operations.
