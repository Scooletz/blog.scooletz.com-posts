---
layout: post
title: "Why did it happen?"
date: 2016-11-24 09:55
author: scooletz
permalink: /2016/11/24/why-did-it-happen/
nocomments: true
image: /img/2016/11/stocksnap_6r7kcxbeee.jpg
categories: ["Design", "Event sourcing"]
tags: ["event driven architecture", "EventSourcing"]
imported: true
---

### **TL;DR**

Do you know that feeling of being powerless? Of being not able to tell why your system acted in a specific way? Of not being able to recognize whether it's a hacker or your system malfunction? Event sourcing, by storing all the events that happened in your system helps a lot. Still, you can improve it and provide much better answers for 'why did it happen?'.

### Why?

The question 'why did it happen?' reminds me of one case from my career. It was a few years back, when I was a great fan of AutoMapper. It's a good tool, but as with every tool one should use it wisely (*if you have only a hammer ...*). I think I stretched it a little to much and landed in a point where nobody was able to tell what the mapping come from. It took only 3 days to provide an extension method *Why* that was showing which mapping will be applied for a specific object.

I'd say, that being able to answer these 'whys' within a reasonable time frame is vital for any project. And I don't mean only failures. When introducing a junior to your team, being able to show how and why things work is important as well.

### Event sourcing

Event sourcing helps a lot. It just stores every business delta, every single change of your domain objects. You can query these changes in any time. You can improve your queries even more by enhancing events with some metadata (which I covered recently in [here](http://blog.scooletz.com/2016/11/14/rediscover-your-domain-with-event-sourcing/)). There's a case though that you should consider to tell if the reasoning is always that easy.

### Dispatching an event

Some of actions on your aggregates are results of dispatching an event. Something happened and another part of your systems turns that into an action. For instance consider the following

When *OrderFinished* then *AddBonusPoints*

Some bonus points are added to an account whenever an order is finished. When looking at an account history, you'll see a lot of events *BonusPointsAdded*. Yes, you could introduce a lot of events like *BonusPointsAddedBecauseOfOrderFinished* but this just leaks the process into your account aggregate. If you don't do it, can you answer the following question

Why *BonusPointsAdded* were appended?

Because somebody added points? Yes, but WHY? It looks that the reason is disconnected.

### Point back

What if following metadata were added to every event that is a result of another event. In this case, what if the following metadata were added to this specific *BonusPointsAdded*

![enhance](/img/2016/11/enhance.png)

Now, when somebody asked Why did it happen you can easily point to the original event. If that was a reason of some process, of dispatching the event you can follow the link again and again to find the original event that was created because of an user action.

### Summary

Links are a powerful tool. If you use it with event sourcing you can get a history of your system that's easy to navigate, follow and reason about.
