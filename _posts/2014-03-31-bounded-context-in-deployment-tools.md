---
layout: post
title: "Bounded context in deployment tools"
date: 2014-03-31 11:00
author: scooletz
permalink: /2014/03/31/bounded-context-in-deployment-tools/
nocomments: true
categories: ["Continous delivery", "Continous Integration", "DDD"]
tags: []
imported: true
---

Recently I've been moving around the topic of a deployment. Imagine a situation you're being given a set of scripts, or script like objects used to deploy a set of applications. The so-called scripts are from the very basic like create-directory to complex, rooted in an organization infrastructure and tooling. Additionally, some of them are defined as groups of other scrips. For example, installing an application service starts with creation of a directory, then binaries are copied and finally, the service is registered.
The scripts are not covered with tests, but are hardened by years of successful usage. One could consider to rewrite them totally, and provide a full blown set of tests. This may be hard, as you throw away all the knowledge hidden behind scripts. Remember, that there were big companies that are no longer here, take [Netscape](http://www.joelonsoftware.com/articles/fog0000000069.html) as example.

I've spent quite a while about considering chef, PowerShell, Pupper even the msbuild with its tasks. What helped me to make up my mind was the famous [Blue Book](http://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215). Why not to consider a set of scrips as a bounded context? Just take a look at the picture provided by Martin Fowler [here](http://martinfowler.com/bliki/BoundedContext.html). Wrap all the older scripts in a context bubble providing mapping, mostly intellectual, to all the terms that are needed to be known outside. It's more than wrapping all old scrips with an interface. There is a need of a real mapping with a glossary to let people which do not want to leave the bubble now exist in it for a while. What tool will be used for the new bounded context communicating with the other? That's an open question. I'll try to choose the best tool, with good enough test support. The only real requirement is the ability to provide the mapping to the old deployment tools' context.

If you want to learn more, just take a loot at the great Eric Evan's videos under this link: http://dddcommunity.org/?s=four+strategies
