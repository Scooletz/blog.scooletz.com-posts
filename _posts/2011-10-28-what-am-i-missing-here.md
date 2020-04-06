---
layout: post
title: "What am I missing here?"
date: 2011-10-28 11:00
author: scooletz
permalink: /2011/10/28/what-am-i-missing-here/
nocomments: true
categories: ["Architecture", "Design", "TDD"]
tags: ["design by feature"]
imported: true
---

If you're in a startup and have a full-time job a the same moment as I do, that's a post for you.

The initial startup pressure and tempo is huge. Focused on the features you can bring to life more and more of them. How often do you load your project, collapse it's whole structure and ask questions:

* What am I doing now?
* How does it influence the rest of the system?
* Is everything I need expressible in the current infrastructure and/or design?
* Is it something, which I know from other projects missing?

It might seem that those opened questions are unneeded, to silly to ask, but from last time I asked them, they became a weekly routine. To show you, I'll give you an example.

I write tests. As you already know, not always unit tests, but... During one of my write test/run/fix error cycles I noticed that it was quite hard to get all the information I needed. There was an assert failing and without debugging, only by viewing logs I had no idea what might have gone wrong. I reopened the project and did 'what am I missing here?' After global review of the whole solution I did found a thing. During all the feature based design I did a silly mistake not providing any logging in the application. You know, these *_if_log_isDebugEnabled_* stuff (take a look in the NHibernate code). It took me no more than 10 minutes to spike it with some console appender and I rerun my tests. Ha! Some components did not log one or two operations and that was it.

It's worth not to loose the (overused phrase) big picture and from time to time, stop providing features and ask these silly, ordinary questions.
