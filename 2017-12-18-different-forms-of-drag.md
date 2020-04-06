---
layout: post
title: "Different forms of drag"
date: 2017-12-18 09:55
author: scooletz
permalink: /2017/12/18/different-forms-of-drag/
nocomments: true
image: /img/2017/12/stencil-default.jpg
categories: ["Architecture", "Design"]
tags: ["architecture", "design"]
imported: true
---

*Have you heard about this new library called **ABC**? If not, you don't know what you're missing! It enables your app to do all these things! I'll send you the links to tutorial so that you can become a fan as well. Have I tested it thoroughly? Yeah, I clicked through demo. And got it working on my dev machine. What? What do you mean by handling a moderate or high traffic? I don't get it. I'm telling you, that I was able to spin an app within a few minutes! It was so easy!*

Drag (physics) is a very interesting phenomenon. It's a resistance of a fluid that behaves much different from regular, dry friction. Instead of being a stable force, the faster an object moves, the stronger the drag is. Let's take a look what kind of drags we could find in modern IT world.



### Performance drag

The library you chose works on your dev machine. Will it work for 10 concurrent users? Will it work for another 100 or 1000? Or, let me rephrase the question: how much of RAM and CPU it will consume? 10% of your application resources or maybe 50%? A simple choice of a library is not that simple at all. Sometimes your business have money to just spin up 10 more VMs in a cloud or pay 10x more because you prefer JSON over everything, sometimes it does not. [Choose wisely and use resources properly](http://blog.scooletz.com/2017/10/30/heavy-cloud-but-no-rain/).

### Technical drag

You probably have heard about the technical debt. With every shortcut you make, just to *deliver it this week, not the next one*, there's a non zero chance of introducing parts of your solution that aren't a perfect fit. Moreover, in a month or two, they can slow you down, because the postponed issues will need to be solved eventually. Recently, instead of *debt* it was proposed to use the word drag. You move on with a debt, but moving with a drag, for sure will make you slower.

### Environment drag

So you chose your library wisely. You know that it will consume a specific amount resources. But you know that it has a configuration parameter, that allows you to cut off some data processing or RAM usage or data storage costs. One example that automatically comes to my mind are logging libraries. You can use the logging level as a threshold for storing data or not. How many times these levels are changed only to *store less data on these poor productions servers*? When this happens, scenario for a failure is simple:

1. cut down the data
1. an error happens
1. no traces beside the final catch clause
1. changing the logging level for one hour
1. <del>begging</del> asking users to trust us again and click one more time

This and similar stories heard tooooo many times.

### Summary

There are different forms of a drag. None of them is pleasant. When choosing approaches, libraries, tools, choose wisely. Don't let them drag you.
