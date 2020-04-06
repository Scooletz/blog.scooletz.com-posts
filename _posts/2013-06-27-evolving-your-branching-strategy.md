---
layout: post
title: "Evolving your branching strategy"
date: 2013-06-27 10:00
author: scooletz
permalink: /2013/06/27/evolving-your-branching-strategy/
nocomments: true
categories: ["Continous Integration", "Git"]
tags: ["branching", "CI", "git"]
imported: true
---

Recently I've been involved in two OSS projects, the first is [ripple](https://github.com/DarthFubuMVC/ripple "ripple"), a part of the Fubu family. The second is my own [extension to Protobuf-net](https://github.com/Scooletz/protobuf-linq "protobuf-linq"). Both are hosted on GitHub which brings a great opportunity to polish Git skills.

I started work on protobuf-linq on the master branch. It was the simplest and fastest way of getting this thing done. Throwing in a continuous built with TeamCity was simple: two steps of building and running tests. Then, I thought about stability of the master branch. Will it be always production ready? After a few conversations, rereading [the great git book](http://git-scm.com/book "Git book") and going through [the nvie's post](http://nvie.com/posts/a-successful-git-branching-model/ "Git branching model") the idea was clear. Make the *master* branch always *a production ready branch* and add another *dev* branch. This would allow separate stable branch which latest version is always good to be built and publish from the development branch. The very same schema is used for instance by[ the Event Store public repository](https://github.com/EventStore/EventStore/branches "Branches of Event Store") and many others.

This simple change, an evolutionary not revolutionary allows to separate *production ready branch* from dev branch malfunctions (history rewrites, errors and so on). Of course you can throw in tags, feature branches and hot-fix branches, but it isn't needed from the beginning for an open source library. This always can be added as the ecosystem grows.
