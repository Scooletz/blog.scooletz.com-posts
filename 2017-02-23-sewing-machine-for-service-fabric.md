---
layout: post
title: "Sewing Machine for Service Fabric"
date: 2017-02-23 09:55
author: scooletz
permalink: /2017/02/23/sewing-machine-for-service-fabric/
nocomments: true
image: /img/2017/02/package_icon.png
categories: ["Service Fabric", "Sewing Machine"]
tags: ["azure"]
imported: true
---

### TL;DR

This is an introductory post for my new Open Source journey project, [Sewing Machine](https://github.com/Scooletz/SewingMachine) that aims at *extending capabilities of Azure Service Fabric services in various ways. It's focused on speed and performance, but also aims at delivering more capabilities build on top of the powerful Service Fabric foundations*.

### Services and actors

Service Fabric provides various capabilities. It can host processes and containers, enabling you to control resources usage on a much more granular level. You can host multiple processes on one VM, if they don't require that much CPU/memory. Beside this, SF provides following models for writing your applications:

* stateless services
* stateful services
* actor framework

These parts will be covered in the forthcoming posts.

### Can we do more

I've spent some time reading official docs and decompiling Fabric sources and found that there are possibilities of building more on top of these strong foundations. Does *more* mean *better*? Possibly yes. I think that in Sewing Machine I'll be able to allocate much less, pin no memory and use better serialization. This is low level. On a higher level I hope for:
* event sourced actors
* better use of secondary replicas (like using them for running processes etc)
* multi actor projections

### Journey, not path

Sewing Machine is in its initial phase. This is another journey project of mine as it's a project based on uncovering what's behind Service Fabric. I will share my findings in following blog posts and work towards reaching the aims above. This also means that version 1.0 is highly unlikely to be published within a month or two. For sure this journey will take a bit more and hopefully findings will be good enough to release it.

I hope you're eager to do some sewing with this fabric. I am.
