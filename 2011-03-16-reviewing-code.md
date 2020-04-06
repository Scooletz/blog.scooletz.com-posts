---
layout: post
title: "Reviewing code"
date: 2011-03-16 22:23
author: scooletz
permalink: /2011/03/16/reviewing-code/
nocomments: true
categories: ["Code review"]
tags: ["code review", "Review Board", "VisualSVN"]
imported: true
---

Recently I've been introducing a code reviewing tool, called [Review Board](http://www.reviewboard.org/). The whole 'review' thing was inspired by **a free copy** of  [Best Kept Secrets of Peer Code Review ](http://smartbear.com/best-kept-secrets-of-peer-code-review/). The aim was to empower my team with a tool, which will allow propagating best practices easily. The other, obvious target, was ensuring quality of a software being developed. In a few next posts, I'll describe the whole process, as well, as a team accommodation.

### The book

The paper version of the book was sent for free. I was astonished that one company can afford such a gift. I do understand that *Smartbear* released their revision tool called [Code Collaborator](http://smartbear.com/products/development-tools/code-review/features/), but... come on. A book for free? It's nice of them. Speaking about this position, it's a must read for anyone who wants to introduce reviewing in his/her team. The book covers some research done in a combat conditions which can be valuable for your manager, if you have problems with incorporating a new thing into your tech toolbox. The numbers show, that reviewing is much more valuable than writing tests. I know, it can harm some zealous TDDers, but you should take into consideration by every developer thinking seriously about his/her craft.

### The tool

As I scanned the *Code Collaborator* pricing I knew, that my company won't cover costs for the whole team. I searched for some replacement and found the Review Board. The application is written entirely in Python and the server part is destined to be installed on a Linux environment. We've managed to make it run in less than one day. Then, the time came to integrate it with VisualSVN hosted on a Windows 2003 server. The VisualSVN has a nice support for post-commit hooks. It passes two parameters to the hook body. Although they can be easily passed to your batch, exe, whatever with a simple property window, it took me a while to get all the needed data to match the [post-review](http://www.reviewboard.org/docs/manual/dev/users/tools/post-review/), as it needs for instance a revision range for which the diff will be generated. Finally, I've got it running.

### The team

As always I tried to be the change. The reviewing after a small number of delays became a habit. Receiving emails helps with this, as each review request is notified with a short email with a request summary. The whole process made the whole team to write better commit comments as no-one wants to be asked (at least, I wouldn't like) "what did you just commit?". It seems, that beside measured factors, the code review empowers a team as a whole. The journey goes on!
