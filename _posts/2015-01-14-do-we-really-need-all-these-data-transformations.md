---
layout: post
title: "Do we really need all these data transformations?"
date: 2015-01-14 11:00
author: scooletz
permalink: /2015/01/14/do-we-really-need-all-these-data-transformations/
categories: ["Design", "Optimization"]
tags: ["design", "mapping", "performance", "zero-copy"]
imported: true
---

Applications have layers. It's still pretty common to see an enterprise application being built with layers like DAL, Business Logic (or Domain), Services, etc. Let's not discuss this abomination itself. Let us rather consider the flow of the data within the application.

### SELECT * FROM

That's where the data are stored. Let us consider a good old-fashioned SQL Server. To get the data from the database you may use ADO (oh no!) or any new ORMs, including the micro ORMs like Dapper or something similar. What you end with is probably some kind of an object, or an object collection. Here's where you start playing with data.

### Mappings

It doesn't matter whether you're using Automapper or map the data on your own. For encapsulation purposes or getting an immutable version of an object it's common to copy its values to a new representation. I know that strings are immutable and will be copied by reference, but you copy them as well.

### Services

So you've got your data mapped to the right model. Now you can return them from your service. Ooops, it's a fancy REST service and you translate the very same data again. Now, because it's a browser asking and you use content negotiation, the data are transformed to JSON.

In onion architectures, you can meet even more transformations between layers, mappings from DTOs to DTOs are quite common. The question, not only from the architecture point of view, but from the performance oriented angle is the same: what are you doing? Why do you want to spend plenty of time to write all these mappings? Why do you want to melt the CPU in never ending mappings? Can you not skip all of these? Why not to store JSON in the database or use a database that supports JSON blobs as a first level citizen (RavenDB, MongoDB) and simply push the content retrieved from the database right to the output stream?

All the thoughts above have been provoked by services I'm creating now. Long story short, they store objects serialized with Google Protocol Buffers. When you access an object from an external system, a service just copies the blob without the deserialization right to the output stream. No deserialization, no allocations, no overhead. Simple and brutally fast.

Next time you come up with an onion design or layers of transformations ask yourself is it worth and if you can pay the price of doing all these mappings.
