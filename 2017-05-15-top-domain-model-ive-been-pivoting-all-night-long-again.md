---
layout: post
title: "Top Domain Model: I’ve been pivoting all night long. Again"
date: 2017-05-15 08:55
author: scooletz
permalink: /2017/05/15/top-domain-model-ive-been-pivoting-all-night-long-again/
nocomments: true
image: /img/2017/04/top_domain_model.jpg
categories: ["DDD", "Event sourcing", "Top Domain Model"]
tags: ["DDD", "Event sourcing", "Top Domain Model"]
---

### TL;DR

In the last entry we noticed that we can model out some problems by selecting a different aggregate an event belongs to. In this episode, we will revisit this idea by referring to the good old-fashioned Twitter and its timelines. How timelines are created? Is everyone treated equally? Is it only aggregates' identities that should be pivoted or maybe our thinking as well?

### Fans in, fans out

The model behind Twitter timelines is quite simple. It's also different for people having 10 followers or 10 millions. The difference is the number of timelines that needs to be updated when Madonna writes a tweet or a @JustAnotherAccountLikingMadonna2019 retweets one of their favorite performer thoughts. Notifying 10 millions people is not that easy and takes a bit longer. Writing to 10 timelines is easy and fast.

### A golden customer

A similar reasoning might be added to a golden customer profile or any kind of dimension that distinguishes an aggregate type in your model. Sometimes, a behavior will be slightly altered, sometimes it will be totally rewired. The most important idea is to don't stick to the initial model. Let yourself ask important questions to distill and clarify it. To discover these golden customers that require different handling, to discover Madonnas that require cargo shipping to deliver tons of their tweets and so on and so forth.

### A new type or not

Sometimes, this reasoning will bring you new types of aggregates, sometimes it will be a process based on a dimension distinguishing a Madonna-like aggregate instance. As always, every model will be wrong, but some of them will be more useful than others.

### Summary

Don't stick to the initial aggregate type. Ask more modelling questions, that can bring you new types of aggregates or processes depending on a specific dimension value of already existing aggregate. Pivot not only identities, but also your thinking.
