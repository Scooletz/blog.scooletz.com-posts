---
layout: post
title: "When stateless services are useless"
date: 2021-07-12 11:00
author: scooletz
permalink: /2021/07/12/when-stateless-services-are-useless
image: /img/2018/11/x.png
categories: ["architecture", "soa", "services", "dapr"]
tags: ["architecture", "soa", "services", "dapr"]
whitebackgroundimage: true
---

Recently, I spent some time on diving into into [dapr](https://dapr.io), the Distributed Applications Runtime. Its premise, visible on the landing page, is simplification of the cloud-native application development. It is a strong one and much aligned with my perception of being Cloud Native. But beside the motto, there's also a book called [Dapr for .NET developers](https://docs.microsoft.com/en-us/dotnet/architecture/dapr-for-net-developers/). It provides a good description of what dapr is and what it provides. There's [one paragraph](https://docs.microsoft.com/en-us/dotnet/architecture/dapr-for-net-developers/state-management) though, that I strongly disagree with

> While each service should be stateless...

It should not and I'll tell you why.

### Let's assume that they should

Let's start with an assumption that services should be stateless. This would require a service to call other services. At least one, maybe two or three. This introduces a few problems:

1. a temporal coupling
1. lowered SLA
1. performance overhead.

The first mentioned temporal coupling introduces a sequential dependency between services. After all, if a service requires another one to respond, it will automatically be dependent on it temporarily, meaning, that it can't execute its functionality without others.

The service has its SLA lowered. If it has no state to decide about the execution it requires it from a third party. This means, that to answer a request, it needs to work on its own as well as have the third party working.

The last but not least is related to the overall performance overhead. If it requires a network hop, a serialization of a request and a response, no matter how good the underlying execution framework is, it will introduce an additional overhead.

### It's a library

Effectively, if a service is responsible only for gathering a few other responses, effectively its work could be captured as a library. Or even simpler, a function (which might be shared as a lib after all). Wrapping it as a service may be reasonable under some circumstances, but usually it's just a waste of resources. What could be a situation where creating a service is a better alternative than having a library shared amongst its potential callers?

1. a secret algorithm
1. specialized hardware
1. heterogenous runtimes

If you come up with a secret algorithm that you don't want to share, wrapping it up as a service could be a reasonable scenario. Especially, if you don't want to deal with licencing and defining what a derivative work is. If this service is used internally though, within a single company, this probably won't be the case.

A service might require a specialized hardware, a GPU or a quantum processor. If this is the case, then having it wrapped as a service might limit the cost of having multiple ones.

The last, which sounds a lot like dapr case, is related to heterogenous runtimes. If there's a Java service, a .NET service and a node app, having an implementation hidden behind a service might help in sharing its implementation.

If none of these are your case, wrapping a part of an implementation as a library might be a good enough solution.

### Evolutionary disadvantage

The last disadvantage is related to the ownership of parts of the system and evolving it into new directions. If a service delegates handling data elsewhere, it's possible that in the end it will become a conformist to another one, that provides a sad CRUD facade. This might limit the ability to hide unneeded details, evolving your service into new directions and seal up the data from third parties accessing them directly. Of course there are scenarios, like data lakes etc. when having a bag of data is ok and is needed. Even in this case, they should come from services that own the master set.

### Summary

It's tempting to use the new and shiny. At the same time it's worth to remember that ownership, including the data ownership is vital fo building healthy long-lasting services.