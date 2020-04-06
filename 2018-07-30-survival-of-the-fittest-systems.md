---
layout: post
title: "Survival of the fittest (systems)"
date: 2018-07-30 08:55
author: scooletz
permalink: /2018/07/30/survival-of-the-fittest-systems/
nocomments: true
image: /img/2018/07/auto-przeglad.jpg
categories: ["Architecture", "Design"]
tags: ["architecture", "design"]
imported: true
---

This time we'll make it right! *Anonymous software architect planning for a big rewrite. 2018*

There's no better feeling that being led through a big rewrite by a prophet saying that this time we'll make it right ;-) Depending on the experience gathered by members of this brave team, one might ask *What will make the difference this time?* or *Are we going to rewrite it again after this rewrite?* At least, I hope they will. In the real world and in IT world there's no way to migrate/rewrite/upgrade something once and just leave it. This statement is even more true when talking about systems, composed from different pieces, services, things. So is there something that might help your system to survive?

### Next generation ++

In the Darwinian words, *survival of the fittest* is an ability to move your genes forward, to be able to pass the natural selection test. Effectively, it's about the next version of you, not about you. The ability of moving forward is crucial for your line of genes to thrive. I'd dare to say, that the same paradigm could be demanded from any system that is meant to thrive. How will it move forward? How will you enable it to release the next version of it with pieces changing slightly between versions.

### Domain Driven Failure

One of the cardinal failures in designing systems is writing them in stone. Whether you use Domain Driven Design or some other approach, if the evolution of the whole is not taken into consideration, this might become a Domain Driven Failure. You can take a while and go to your business and ask what was the last time the rules for running your company changed. Is it a week, a month? How about applying GDPR or other legal requirement? I'm not saying that you should design a multi-plugin modular micro-serverless mubmo jumbo. I'm saying that choosing some angles of freedom is important and vital.

### Contracts and API

When thinking about what might change, I think about public contracts and API. These might be message contracts, REST API, gRPC  protos or other schemas that you use for communication. Being able to version it properly, possibly supporting blue-green deployment for breaking changes to let client move to the new version when they can, is the way to go. This might be also shifted to the infrastructure like:

1. API Gateways - with custom rules for routing requests
1. Brokers - with some transformations etc.

I'm not a fan of asking for changing rule X in a set of thousands of rules, so I want to present another alternative as well.

### 1.0 & 2.0

You could host two versions of the API in the same service. Once the API call is translated to your current version, you'll have no problem with versioning internal state (database, etc). This means paying some tax in terms of the code and hosting two version, still, it might address some infrastructure-related pain points.

### Message routing

When you use messaging, the versioning gets even more interesting. Let's imagine that, because of GDPR, you moved all the user data to the other service (The One That Should Be Named). Now, in the old User service, you might still accept the message with the request for changing some personal data and simply reroute it to the new one. This enables another level of freedom when versioning services. It's the called service that can help to move functionality to the other one. The client, once it has it's routing migrated, will call the new one.

### <del>Revolution</del> Evolution

Letting your services, and, because of the former, systems evolve, will help you a lot. It lowers the probability of the big <del>failures</del> rewrites and help you to make more changes in a stable manner. Yes, it costs, as every thing you take care of, but it helps a lot, if you want to keep your system healthy.
