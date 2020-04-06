---
layout: post
title: "Code reviews"
date: 2014-02-12 10:00
author: scooletz
permalink: /2014/02/12/code-reviews/
nocomments: true
categories: ["Code review"]
tags: []
imported: true
---

Recently, I've been involved in a discussion about code reviews. It made me remember a code review board that I introduced in one of my previous projects. Beside that I had to clarify positive aspects of code reviews. Verified by experience the most important point of doing code reviews with a tool is total asynchrony of the process. The reviewer can easily go through the diff, marking lines, changes in a given time selected by himself (or scheduled via team rules). The second point would be the artifact left after this process. It's not a discussion over a coffee or an email in a private inbox. Once it's published in a tool, it's visible part of the project, a manifestation of the change in the code. Isn't it great?

The list of possible tools to use:

* [https://github.com](https://github.com) with it's possibility to comment over pull requests and commits.
* [http://www.reviewboard.org/](http://www.reviewboard.org/) I used this tool, it's simple and easy to go with. It's free!
* [Crucible](https://www.atlassian.com/software/crucible/overview) an Atlasian tool. I haven't used it so far, but hope to do it in a while

Getting asynchronous, public, strongly connected with code discussion over the project. This will make your code thrive and the knowledge spread across the team. Trust me.
