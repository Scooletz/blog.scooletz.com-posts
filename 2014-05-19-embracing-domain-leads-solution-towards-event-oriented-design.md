---
layout: post
title: "Embracing domain leads solution towards event oriented design"
date: 2014-05-19 11:00
author: scooletz
permalink: /2014/05/19/embracing-domain-leads-solution-towards-event-oriented-design/
nocomments: true
categories: ["DDD", "Git"]
tags: ["domain modelling", "EventSourcing", "git"]
imported: true
---

One of the most powerful aspects of git is it's simplicity. One can easily read [the object chapter of the Git book](http://git-scm.com/book/en/Git-Internals-Git-Objects) during one afternoon and learn, that git stores nothing more than snapshots of the current state of the items added to the repository. If an item is repeated in multiple commits with no changes, it's referred under the same SHA1 value and it doesn't have to be stored twice. This decision is well explained by Linus in [here](http://permalink.gmane.org/gmane.comp.version-control.git/217). The major point of this explanation is that having an algorithm finding changes, narrow enough and well described like *method declaration moved*, etc. can be hard and costly. To store a snapshot of the state is much easier and let's you run your algorithms much later. This allows algorithms evolutions working on the unmodified version of the state at any time.
What git does is storing the every state you commit. The commit object contains a reference to a tree object, which consists of other objects. This results in storing state of any commit in the entire repository history. This means that **git never overrides the state**. All it does is adding more and more with pointers to the parent states/commits. This allows you to run any tool/algorithm through the entire history of any branch.
This considerations rooted in the git repository design can imply following paradigms to modelling.

### State driven modelling

It's simple to store state. All you've got to do is to serialize all the data and put into the store, but... how many times you've written a system which performs updates? What about the earlier state? Is it preserved? Or maybe it's overridden? I can hope that previous values are stored in a some kind of an audit log, but it's an audit log, not your previous state, isn't it? It's not the same. Nathan Marz is discussing the fragility of updates in his talk [here](http://www.infoq.com/presentations/nosql-cons). Maybe storing a new state with a link to the previous one (no audit log, just the old value of the state) isn't that bad after all.

### *Changed* events

The second take on modelling would be embracing the changes with *___Changed* events. You know a property/getter changed it's value and it's good to audit it. Unfortunately it can be met in solutions which requires audit logs. Storing a common '*name - old value - new value*' tuple is easy. It may be not that simple to deal with any domain changes, or get your new algorithm run through the state of a given entity through entire history but it's easy. I'd consider it a poor man solution for a person which doesn't want to invest his/her time in learning the domain. One can audit anything with this kind of paradigm. It's all text after all, isn't it?

### Event sourcing

The last take is event sourcing capturing the business events, which applied onto the previous state lead to the next one. This is also mentioned by Linus, when he talks about clever algorithms calculating perfect deltas. To get one, to get the event and the transition/change it emits when applied, it takes a lot of investment into understanding the domain. I can imagine that perfect events for git repository of C# project would contain events like:

* method renamed
* method moved
* functionality added

Of course it is/may be impossible to provide this kind of information retrieval but it shows the way, that well understood and enriched with right events' types domain can be minimalistically described with a set of events.
