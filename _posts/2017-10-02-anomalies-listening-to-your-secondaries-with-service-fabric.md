---
layout: post
title: "Anomalies: Listening to your secondaries with Service Fabric"
date: 2017-10-02 08:55
author: scooletz
permalink: /2017/10/02/anomalies-listening-to-your-secondaries-with-service-fabric/
nocomments: true
image: /img/2017/10/anomalies.jpg
categories: []
tags: ["anomalies", "Service Fabric"]
imported: true
---

This is the second post in the series describing different anomalies you may run into using modern databases or other storage systems.

### Just turn this on

This story has a similar beginning as the last one. It starts when one of developers working on a project built with *ServiceFabric* finds this property [ListenOnSecondary](https://docs.microsoft.com/en-us/dotnet/api/microsoft.servicefabric.services.communication.runtime.servicereplicalistener.listenonsecondary?view=azure-dotnet) and enables this feature. After all, if now every node in my cluster can answer queries sent by other parts, that should be good, right? I meant, it's even more than good! We're faster now!

### Replication

To answer this, we need to dive a bit deeper . We need to know how Service Fabric internal storage works. Service Fabric provides a clustered storage. To ensure that your data are properly copied, it uses a replication protocol. In every single moment, there's only **one active master**, the copy accepting all the write and read operations, replicating its data to all the **secondary replicas**. Because of various reasons, replicas that data are copied to, can be not always up to date. To give an example, imagine that we sent three commands to Service Fabric to write different pieces of data. Let's take a look at the state

* master: cmd1, cmd2, cmd3
* replica2: cmd1, cmd2,
* replica3: cmd1, cmd2, cmd3

Eventually, *replica2* will receive the missing *cmd3*, but depending on you hardware (disks, network), there can be a constant small lag, where it has some of the operations not replicated yet.

Now, after seeing this example of how replication works and noticing that the state on replicas might be occasionally stale, can we turn on **ListenOnSecondary** that easily?

### It depends (TM)

There is no straight answer to this. If your user first calls an action that might result in a write, and then, *almost-immediately,* queries for the data, **they might not see their writes**, which are replicated with some lag.

If your writes are not followed with reads, and you always cheat by updating the view for the user as it would be, if data were read from the store, then, you might not run into a problem.

Unfortunately, before switching on this small flag, you should think about concerns I raised above.

### Wrapping up

Unfortunately for us, we've been given a very powerful option, configured with a single call to a method. Now, we can enable reading potentially stale data to gain bigger query throughput. It's still up to us, whether we want to do it and whether we can do it, being given the environment and the architecture our solution lives in.
