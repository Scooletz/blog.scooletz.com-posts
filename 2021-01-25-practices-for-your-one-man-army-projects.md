---
layout: post
title: "Practices for your one-man-army projects"
date: 2021-01-25 11:00
author: scooletz
permalink: /2021/01/25/practices-for-your-one-man-army-projects
image: /img/2021/01/one-man-army.png
categories: ["programming", "practices"]
tags: ["programming", "practices"]
whitebackgroundimage: true
---
There are times where you have a team that is helpful and willing to review your PRs, discuss all the internals of a project and simply support you in making it work. There are also your own projects where you do things alone following the proverbial "one man army" mantra. Just imagine writing the **next big thing** without the whole crew and all the resources that are at your disposal when working somewhere. When talking about a project like this, I find the following practices really helpful. I'm far from stating that the determine the success of the project itself, but for sure they helped me a lot when working on various things.

### Keeping multiple problems in your head

It's better to have multiple problems than just one. Let's define a problem a bit better. By a problem I mean a question or a task that is hard to solve in an efficient manner, but doesn't have to be solved right now. For example

> Imagine that you're writing a data processing platform that deals with loads of data. It's ok to introduce a simple suboptimal solution for, let's say, indexing, and then keep the indexing problem as open.

By keeping problems open, I don't mean counting the technical debt or writing down long notes about the specific challenge. What I mean is keeping them open in your head as they may be solved later on, when you're inspired by music, a walk or maybe a beautiful sunshine.

### Testing

When dealing with software testing is one of the most fundamental tools you can use. I'm far from being a test-addict and seeing everything as a never ending loop of red-green-refactor. What I mean by testing is using a full pallette of tests to help you move on an address the missing parts. For instance, in one of my projects

> I had a suite of unit tests that had a good coverage. To make sure that I'm addressing all the things I introduced a few smoke tests, that verify whether the core functionalities work from end to end. One of them became red after introducing some changes. I was fine with this for while but finally, it was the one that helped me to find the bug and fix the problem.

Using a variety of tests can help you a lot by measuring and providing different perspectives of your product.

### Inspection windows

Even when working on the most top secret project that will change the world, you're not alone. What you can do is to wisely select an inspection window to your system and ask a person or a group of people about this specific part. Of course selection may reveal something about your work, but if the windows is selected properly it can be beneficial for learning of both sides of the conversation. Let's consider the following example

> You're working on a sliding window query tool that will group entries with some level of fidelity and aggregate them in time. Your idea is to hash entries with a non-cryptographic hash and then group them by its value. What you can do is to ask about non-crypto hashes, sliding window algorithms or even tools, as it may show the existence of a better tool that you're working on.

Inspection windows should be chosen wisely. Additionally, you should ask yourself how likely is it for you to boast about your hidden agenda. Depending on your talkativeness, you should reconsider the way you convey the message. Maybe a written form is much better as you can edit out what's not needed?

### Whitepapers and books

Most of the problems are solved, even the non-trivial ones. Searching for books and whitepapers dealing with your topic is vital and provides you with an opportunity to either use what's already proven in a battle or re-invent standing on shoulders of giants. Unfortunately, people, including scientists, are unwilling to document their failures, so not finding your approach doesn't mean that you invented something. The survivorship bias may be kicking in and you may follow the undocumented path of these who did not survive.

### Summary

The approaches described above helped me many times when working on different ideas. They are not unique to one-man-army projects, but I find them really helpful when working on something in a solo mode.
