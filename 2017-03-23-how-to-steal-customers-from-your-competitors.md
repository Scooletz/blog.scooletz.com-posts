---
layout: post
title: "How to steal customers from your competitors"
date: 2017-03-23 09:55
author: scooletz
permalink: /2017/03/23/how-to-steal-customers-from-your-competitors/
nocomments: true
image: /img/2017/03/stocksnap_j04bj1vcgk.jpg
categories: ["Business"]
tags: []
imported: true
---

### TL;DR

I've seen this pattern more than a few times during last few years. It looks that the approach I describe below is quite handy when trying to steal clients from your competitors in IT landscape.

### I look the same but I'm better

Have you heard that you can use MongoDB driver to connect to [Azure DocumentDB](https://docs.microsoft.com/en-us/azure/documentdb/documentdb-protocol-mongodb)? Just like that one can easily swap it's database and use DocumentDB without changing a line of its code. If your app weren't cloud native and as one of the strategies you were considering was running MongoDB on your own in the cloud, you don't have to do it any longer. You can use this offering of the Database as a Service and simply make your app cloud-ready in a matter of seconds (just change the connection string).

Have you heard about the [ScyllaDB](http://www.scylladb.com/)? It's a functional port of Cassandra database. It's  was written in C++ with a custom SEDA-like architecture, user-level network drivers and a lot of understanding of the mechanical sympathy. How does it work? It supports the same network protocol, it supports the same file structure on your disk. Does it look similar? Yes, you can use it as a drop-in replacement. No migrations, not a single line of your code rewritten. Isn't it great?

### API parity, feature parity

It's often said that the feature parity can hurt your business. If you can be compared you will be compared to others and eventually, the better/cheaper will win. What about API parity? What about ScallaDB that can support the same workload on 10x smaller number of servers? What about DocumentDB that is served as a service and additionally has its amazing indexing algorithm? They strive for this comparison, especially when they guarantee no-op switch using not the feature parity but the API parity.

### Summary

Mimicking and setting free customers from a vendor lock-in looks like an interesting and valuable vector of attack for products that offer something more, under the same layer of API. I think, that especially in the public cloud sector, we'll see it more and more.
