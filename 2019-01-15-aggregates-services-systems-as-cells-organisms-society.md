---
layout: post
title: "Aggregates, services, systems as cells, organisms, society"
date: 2019-01-15 09:55
author: scooletz
permalink: /2019/01/15/aggregates-services-systems-as-cells-organisms-society/
image: /img/2019/01/loupe.png
categories: ["DDD"]
tags: ["DDD"]
whitebackgroundimage: true
---

Does a team colleague reads your DNA daily? Do you know what's going on in your cell number A345BC345T343? Is there a single cell in your body that can ask your whole body to stop for a second? I hope the answer to all these questions is NO. Let's take a look at all these different levels of complexity and hiding it.

### Cells a.k.a aggregates

If we take a look at a cell, it might be a good candidate for representing an aggregate. Why would it be?

1. it has its boundary (even physical one)
1. it does not reach to other cells' interiors, so it keeps others' boundaries safe
1. it encapsulates its state, providing a limited way of interacting with it

Similar to aggregates, cells have types, but there are many cells of every type.

### Services a.k.a. organisms

Services defines higher-order boundaries. We could compare them to organs, or even, organisms, like people. Cells within them interact and build a structure that supports life. It's unlikely that every service we create will support life, but hey, let me push this metaphor a little bit further.

The interesting aspect of being an organism is that it can interact with other organisms. Even more interesting is the fact, that these interactions are not constant and a single human being can have different interactions in different times of a day, a month, a year. You work, spending some time with your team, you play and laugh with your family, etc.

### Systems a.k.a social groups

All the interactions between people, creating some kind of social groups, could be named as systems. There no single system running it all. It's rather different systems consisting of potentially the same parts, just configured in different ways. Systems are made of its parts. That's it.

### Is there more

Of course there is. In life, we could draw as many lines as needed. Tissue, organs, even parts of a cell could be treated. As always, by bringing a metaphor of a system, service or an aggregate we want to distill only things that are crucial for the model to be useful.

I wish you finding all the needed boundaries.
