---
layout: post
title: "This noisy neighbor"
date: 2018-05-17 08:55
author: scooletz
permalink: /2018/05/17/this-noisy-neighbor/
nocomments: true
image: /img/2018/05/neighbour.jpg
categories: ["Personal"]
tags: ["personal"]
imported: true
---

Hopefully, you're not one of them. Hopefully, you don't have one. For sure you heard about them. The noisy neighbors. They party a lot, they grill things as if they want to create an artificial fog and they are loud. The only thing you can do, is to arrange your apartment in a way that the sleeping room is far from their apartment. If you live in a house, you can plant a few trees to just make the fence a bit more dense. You follow the pattern: you pay the price for making them less noisy.

The same asymmetrical rule can be applied in the IT world in so many dimensions. You can think of it when working with a "noisy unit". You can pay the price when you need to use this 3rd party service that doesn't work well. It might be the crappy legacy system that everything should connect to.

Fortunately for the code related scenarios, we can put a facade before the noisy neighbor. We can even use the strangler pattern, narrowing all the cases where the crappy part is used and, eventually, remove it, replacing it with a better alternative. The most important thing is to notice the noisy agent and act accordingly. Otherwise, the price, will be paid in an asymmetrical way - much too many times, in much too many ways.
