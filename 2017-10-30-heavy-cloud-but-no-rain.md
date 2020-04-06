---
layout: post
title: "Heavy cloud but no rain"
date: 2017-10-30 09:55
author: scooletz
permalink: /2017/10/30/heavy-cloud-but-no-rain/
image: /img/2017/10/stencil-default.jpg
categories: ["Azure", "Cloud"]
tags: ["azure", "cloud", "functions"]
imported: true
---

Recently I've been playing with Azure Functions. Probably, I should use a bigger word than "playing", because I implemented a full working app using only functions. 4$ , that was all that I needed to pay for the whole month after running some synthetic load through the app. I spent a few additional hours just to make it 3$ next month. You could ask, what's the reason. Read along.

### Heavy cloud

Moving to cloud is getting easier and easier. With the new backendless (let's stop calling it serverless) you can actually chop your app into pieces and pay only when they are run. More than this. You've got everything monitored, so effectively you can see where you spend your money. If you're crazy enough, you could even modify the workflow of your app, to make the heavy work at the end of a chain, to postpone it till a user really needs it. Still, these optimizations and thinking don't seem to be popular this days (or at least I haven't seen it popping up that frequently).

### But no rain

The synthetic load I used to stress the app was simulating a single not that active user. A real usage would be probably much higher, with the price being much bigger. Effectively, instead of treating this optimizations as 1$ only, I could say that I cut the cost by 25%. Now this was only an experiment, but think about it again. A dummy, fast implementation was cheap, but with some additional work I could have done it more profitable. If a price for the cheapest option would be 5$, these are some real gains. These are differences that can make you either profitable or bankrupted.

### Make it rain

In past years developers weren't dealing with money. Servers were there, sometimes faster, sometimes slower. Databases were there, spending countless hours on running our not optimized queries. This time is ending now. Our apps will be billed and it'll be our responsibility to earn money by making them thinner and faster. Welcome to the cost aware era of software engineering.
