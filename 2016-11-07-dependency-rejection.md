---
layout: post
title: "Dependency rejection"
date: 2016-11-07 08:55
author: scooletz
permalink: /2016/11/07/dependency-rejection/
nocomments: true
image: /img/2016/11/no-1532844_640.jpg
categories: ["Architecture", "Design", "Design patterns"]
tags: ["architecture", "design", "design patterns"]
imported: true
---

**TL;DR**

*This values must be updates synchronously* or *we need referential integrity* or *we need to make all these service calls together* are sentences that unfortunately can be heard in discussions about systems. This post provides a pattern for addressing this remarks

### The reality

As Jonas Boner says in [his presentation](https://www.youtube.com/watch?v=9gLrCPVrXo4) *Knock, knock. Who’s there? Reality.* We really can't afford to have one database, one model to fit it all. It's impossible. We process too much, too fast, with too many components to make it all in one fast call. Not to mention transactions. This is reality and if you want to deny reality good luck with that.

### The pattern

The pattern to address this remarks is simple. You need to say **no** to this absurd. **No**. **No**. **No**.  This **no** can be supported with many arguments:
* independent components should be **independent**, right? Like in-de-pen-dent. They can't stop working when others stop. This dependency just can't be taken.
* does your business truly ensures that everything is prepared up front? What if somebody buys a computer from you? Would you rather say **I'll get it delivered in one week** or first double check with all the components if it's there, or maybe somebody returned it or maybe call the producer with some speech synthesizer? For sure it's not an option. This dependency just can't be taken.

You could come up with many more arguments, which could be summarized simply as a new pattern in the town, **The Dependency Rejection.**

Next time when your DBA/architect/dev friend/tester dreams about this shiny world of a total consistency and always available services, remind them of this and simply **reject dependency** on this unrealistic idea.
