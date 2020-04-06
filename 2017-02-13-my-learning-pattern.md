---
layout: post
title: "My learning pattern"
date: 2017-02-13 09:55
author: scooletz
permalink: /2017/02/13/my-learning-pattern/
image: /img/2017/02/stocksnap_u0y3sc9z42.jpg
categories: ["Personal"]
tags: ["personal"]
imported: true
---

### TL;DR

This is a personal entry about my framework that I use subconsciously for learning new things. I've applied it recently when learning about Azure Service Fabric, and then I realized what I did. Next, I thought that it could be useful to share the way I learn new frameworks/tools/libraries. Are you focused?

### Azure Service Fabric is the new black

Azure Service Fabric is a new environment for creating distributed apps for both, cloud and on-premise environments. It provides a lot of tooling with different behaviors (stateful, stateless services, actors). I've been working with Azure Storage Services for some time and two weeks ago I started learning Service Fabric. From zero (beside my distributed systems know-how and some exp with a public cloud).

### Reading docs till you know it all

One may say that you don't need to know everything about a technology to use it. Yes, you don't. My aim is not to use a technology but to know it, to immerse into it, to embrace all the patterns behind it natively. Once I got it, it's like a new language.

In case of Service Fabric it was simple. There is an official [doco page](https://docs.microsoft.com/en-us/azure/service-fabric/) which takes a few hours to read. I did it. In my opinion you truly want to learn vocabulary, patterns, properties to embrace one thing. That's why I read some of the pages more than once.

### F12 till it hurts

It wasn't a point where I was ready to write my first program. I was following the framework code going with F12 (go to the implementation) as deep as possible. Sometimes it requires taking notes, but moves you forward with an amazing speed. That's how I get the real knowledge how it works.

### Write an extension, no dummy sample

The final step is to either send a PR or maybe write a simple extension to the framework. No mambo-jumbo playing with a dummy sample running one actor. I need to prove that foundations are strong. That's how I rebuilt Actor's part of the Service Fabric to be somewhat faster.

### Summary

This is the approach I use for learning new technologies. Total immersion and getting as deep as possible, as soon as possible. Sometimes it hurts, but still, gains are tremendous!
