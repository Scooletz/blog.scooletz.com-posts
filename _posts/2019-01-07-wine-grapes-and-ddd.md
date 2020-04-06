---
layout: post
title: "Wine, grapes and DDD"
date: 2019-01-07 09:55
author: scooletz
permalink: /2019/01/07/wine-grapes-and-ddd/
image: /img/2019/01/grape_3.jpg
categories: ["DDD"]
tags: ["domain driven design", "domain modelling"]
imported: true
---

A while ago I sealed up a few bottles of home made wine (created only for oenology purposes ofc.) Somehow, maybe because of winter's time, my memory resurfaced these moments and connected them with something I'm working now currently, which is connected with Domain Driven Design. Is there any way the process of producing wine and DDD could be connected? Let's take a look.

### Just grapes

Before smashing fruits, I had them cleaned and separated from the bunches. There were just soaked in water and waiting. There was no connections visible.

![grape_1](/img/2019/01/grape_1.jpg)

### They have color

Stating that all the grapes are the same would be a huge misunderstanding, especially, if you use different kinds of grapes. Clearly, they could be grouped by some criteria, like color, shape, sweetness (nope, I don't bite every single grape :P). This would create a kind of categorization of the grapes.

![grape_2](/img/2019/01/grape_2.jpg)

> If you're interested in Domain Driven Design, aggregates and modelling domains, take a look at [Master of Aggregates](https://masterofaggregates.com/).

### This might look like this

The last but not least, is imaging what bunches could look like. Again, seeing through the criteria lenses we can guess or spend more time on investigating what kind of model might be applied to make sense out of it. It's worth to notice that grapes of the same kind may come from the same tree, but different bunches, so there's no reason to try to imagine extremely large bunches. Match them by criteria but do not load everything to single thin branch. It wouldn't be able to hold it up!

![grape_3](/img/2019/01/grape_3.jpg)

### This is the model

At the end, the final picture represents a model. Something that might be true, something that might be summarized as "this is how I think it works".

In this example I inverted the process of making wine, which hopefully nobody will try to apply. Still, I think it is an interesting metaphor for finding and distilling (pun intended) your models.
