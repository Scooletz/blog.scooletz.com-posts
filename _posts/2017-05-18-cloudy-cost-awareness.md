---
layout: post
title: "Cloudy cost awareness"
date: 2017-05-18 08:55
author: scooletz
permalink: /2017/05/18/cloudy-cost-awareness/
nocomments: true
image: /img/2017/05/stocksnap_ita18fxibl.jpg
categories: ["Cloud"]
tags: ["azure", "cloud"]
imported: true
---

### TL;DR

Our industry was forgiving, very forgiving. You could not put an index, run a query for 1 minutes and *some users of your app would be disappointed*. If you were the only one on the market or delivered banking systems, that was just fine as you'd no loose clients because of it. The public cloud changes it and if you can't embrace it, you will pay. You will pay a lot.

### Pay as you crawl

If you issue a query scanning a table in Azure Table Storage, every entity you access will be counted as a storage transaction. Run millions of them and your bill will be increased. Maybe not that much, but it will.

If you deploy a set of services as Azure Cloud Services, each of them consuming just 100MB of memory, your VMs will be undersaturated. You'll pay for memory you don't use and CPU that just sits in the rack that hosts your VM.

### Design is money

Before public cloud, all these inefficiencies could be more or less tolerated, but were not that easy to spot on. Nowadays, with a public cloud, it's the provider, the host that will notice them and charge you for them. If you don't design your systems with the awareness of the environment, you will pay more.

### Mitigations

This is not black or white situation. It never is. You'll probably be able to *dockerize* some parts of your app and host it inside of Service Fabric cluster. You'll probably be able to use CosmosDB and its autoindexing feature to *just fix the performance* for lookups in your Azure Storage Tables. There's a lot of ways to mitigate these effects, but still, I consider a good appropriate design as the most valuable tool for making your systems not only well performing and effective but, eventually, cheap.

### Summary

Don't throw your app against the wall of clouds and check if its sticks. Design it properly. Otherwise, it may stick in a very painful and cost ineffective way.
