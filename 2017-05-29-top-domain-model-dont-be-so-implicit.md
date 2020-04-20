---
layout: post
title: "Top Domain Model: Don't be so implicit"
date: 2017-05-29 08:55
author: scooletz
permalink: /2017/05/29/top-domain-model-dont-be-so-implicit/
nocomments: true
image: /img/2017/04/top_domain_model.jpg
categories: ["DDD", "Event sourcing", "Top Domain Model"]
tags: ["DDD", "Event sourcing", "Top Domain Model"]
---

### TL;DR

Make things implicit (more) explicit. This is the mantra we often hear, but when touching the reality of software development, often forget, because of the magic beauty of technology, tooling, etc. Should we repeat this, when modelling domains as well?

### Auditing

It doesn't matter whether you apply DDD or CRUDdy. It's quite often to hear that a system should provide a some kind of audit. Then we provide a table like History or Audit, where all the "needed" data are stored. What would be the columns of this table? Probably things like:

* user
* operation
* field changed
* value entered

or something similar. Again, depending on the approach these data could flow through the system as some kind of tuple, before hitting the storage.

When dealing with a business process that uses an accountant id, just for registering who's responsible for tax calculations, would you "reuse" the user from a tuple above, or would you rather create a variable/field that is named *accountantId*, even if the values in *accountantId* and *user* would be the same? What would be your choice?

### Event sourcing

*ProperyChanged*, is one of the worst events ever. Yes, it's so beautifully generic and so well describing the idea of event sourcing, grasping changes in the model. It can be gathered implicitly, just by comparing an object before and after applying an action. At the same time it's so useless, as it provides no domain insight and could be blindly used in any case. This approach is often referred to as *property sourcing* and is simply wrong.

The power of event sourcing, or modelling events at all, is grasping real business deltas of the domain. No playing mambo-jumbo with the tooling and implementing the new minimal change finder. Being explicit is the key to win this, to design a better model.

### Summary

There are many more examples of explicit vs implicit and abusing modelling by playing not with the domain but with the tooling. When modelling, analyzing, ask yourself these questions. Should I really reuse it? Should I really make it generic? Quite often the answer will be no, and trying to do this, might hurt your model, and effectively, you.
