---
layout: post
title: "Protect development at all cost"
date: 2015-12-04 11:00
author: scooletz
permalink: /2015/12/04/protect-development-at-all-cost/
categories: ["Uncategorized"]
tags: []
imported: true
---

Some time ago I wrote a few entries about my attitude to the GitFlow and [a small adjustment](http://blog.scooletz.com/2015/05/04/gitflow-and-code-review/) that can greatly reduce the possibility of breaking development/release branches. The main reasoning was following: review, build your merge commits in your feature branches, then using *fastforward-only* merge apply it on develop. Having this applied should keep your development, healthy, right?

Not at all. The final step is pushing changes. And it's a manual step, as ending the feature is. If that's a manual step, there's a bunch of things that can wrong. Yes, you can push quick-fixes, fixups, minor-improvements to development and say to your whole team "it'll be there in a sec", but isn't it easier to just don't break things? Having that said, is possible to be guarded against push that isn't proper? Yes, and my automation for this is simple:

* use git prehooks
* add check against TeamCity API if the build is ok
* add check against a code review tool
* allow push only if the checks are successful

In the following posts I'd present how to

* create git hooks, preferably using PowerShell ("You won't be able to run on Linux oooooh nooooo!")
* successfully get statuses and consume TeamCity API
* combine this to revoke commits that might hurt *development* branch

Stay tuned!
