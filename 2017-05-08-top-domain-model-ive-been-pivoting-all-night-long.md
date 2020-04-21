---
layout: post
title: "Top Domain Model: I've been pivoting all night long"
date: 2017-05-08 20:55
author: scooletz
permalink: /2017/05/08/top-domain-model-ive-been-pivoting-all-night-long/
nocomments: true
image: /img/2017/04/top-model.png
whitebackgroundimage: true
categories: ["DDD", "Event sourcing", "Top Domain Model"]
tags: ["DDD", "Event sourcing", "Top Domain Model"]
---

### TL;DR

This is the next entry in the Top Domain Model series of loosely coupled posts describing different modelling approaches. In [the last one we covered a temporal dimension](http://blog.scooletz.com/2017/05/01/top-domain-model-im-temporal), it's time to consider what is an aggregate and where an event belongs to.

### Endorsements

We all love receiving endorsements on LinkedIn, don't we? How would you model such a feature? How would you draw a boundary of the aggregate? Would one aggregate be sufficient enough? Let's first reason about a single aggregate provided by a user.

### Endorsements profile per user

You could easily imagine providing an *Endorsements* aggregate per user. It could be visualized as a REST subresource of a user like

> /user/1231423/endorsements

This could be implemented in a simple way and would handle properly situations where endorsing somebody is not that frequent. In other way, that the probability of having two users endorsing the third at the same time is quite low. What if it was a famous CTO who just created his profile? What if the number of endorsements would be bigger that 100/s? This model would not hold? What would it then?

### Pivot to the rescue

When modelling, answering what's the subject and what's the action is not that straightforward. You may say that:

> An user is endorsed by another

which means that user's endorsements are modified by another user. But you could also rephrase this to

> An user endorses another

Which could be modeled as endorsements storing another users tags in YOUR aggregate. This model, even with a lot of people endorsing is easily scalable: everyone is applying endorsements for other users in their own aggregate. You won't endorse more than 10 skills a sec, will you?

This approach requires to create a separate, reactive view applying all the events to the read side, so that the famous CTO gets their skills displayed. But this, as it can be eventually consistent, is much easier than overcoming a high throughput scenario for writes.

### Summary

When dealing with high volumes of operations ask, if the model can be pivoted? Maybe, selecting another owner of business operation, or even, modelling a new type of an aggregate, can help you deal with requirements you need to satisfy.
