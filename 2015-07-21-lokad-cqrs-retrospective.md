---
layout: post
title: "Lokad.CQRS Retrospective"
date: 2015-07-21 11:00
author: scooletz
permalink: /2015/07/21/lokad-cqrs-retrospective/
nocomments: true
categories: ["Architecture", "DDD"]
tags: ["architecture", "cloud", "CQRS", "Lokad"]
imported: true
---

In [the recent post](https://abdullin.com/lokad-cqrs-retrospective/) Rinat Abdullin provides a retrospective for [Lokad.CQRS framework](https://github.com/Lokad/lokad-cqrs) which was/is a starting point for many CQRS journeys. It's worth to mention that Rinat is the author of this library. The whole article may sound a bit harsh, it provides a great retrospection from the author's and user's point of view though.

I agree with the majority points of this post. The library provided abstractions allowing to change the storage engine, but the directions taken were very limiting. The tooling for messages, *ddd console,* was the thing at the beginning, but after spending a few days with it, I didn't use it anyway. The library encouraged to use one-way messaging all the way down, to separate every piece. Today, when CQRS mailing lists are filled with messages like 'you don't have to use queues all the time' and CQRS people are much more aware of the ability to handle the requests synchronously it'd be easier to give some directions.

The author finishes with

*So, Lokad.CQRS was a big mistake of mine. I'm really sorry if you were affected by it in a bad way.*

*Hopefully, this recollection of my mistakes either provided you with some insights or simply entertained.*

which I totally disagree with! Lokad.CQRS was the tool that shaped thinking of many people, when nothing like that was available on the market. Personally, it helped me to build a event-driven project (you can see the presentation about this [here](https://www.youtube.com/watch?v=a_MsaM89LDA)) based on somehow on Lokad.CQRS but with other abstractions and targeted at very good performance, not to mention living documentation built with Mono.Cecil.

## Summary

Lokad.CQRS was a ground breaking library providing a bit too much tooling and abstracting too many things. I'm really glad if it helped you to learn about CQRS as it helped me.  Without this, I wouldn't ask all the questions and wouldn't learn so much.

The provided retrospective is invaluable and brings a lot of insights. I'm wishing you all to make that kind of ground breaking mistakes someday.
