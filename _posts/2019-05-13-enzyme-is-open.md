---
layout: post
title: "Enzyme is open"
date: 2019-05-13 08:55
author: scooletz
permalink: /2019/05/13/enzyme-is-open/
nocomments: true
image: /img/2018/10/enzyme.jpg
whitebackgroundimage: true
categories: []
tags: ["memory", "performance", "serialization"]
imported: true
---

My serialization project Enzyme is open for educational purposes. You can find its sources in [https://github.com/scooletz/enzyme](https://github.com/scooletz/enzyme). If you haven't heard about it, in these two posts: [post 1](http://blog.scooletz.com/2018/10/01/enzyme-an-experimental-serializer-for-modern-net/) and [post 2](http://blog.scooletz.com/2018/10/08/enzyme-digesting-even-faster/) I described some findings and reasoning behind its design. Enzyme repository will not be maintained, it's just a snapshot released under Apache. To make it easier to understand and to follow, I tried to do my best and describe all the principles that I used for building it. You can find everything in the [Readme.md](https://github.com/Scooletz/Enzyme/blob/develop/README.md).

### Bits of history

Initially Enzyme was not meant to be a serializer. Actually, it wasn't meant to be at all. It was just a better way of storing millions of data. Then, after extracting bit by bit I was able to wrap it in this asymmetric serializer form.

The asymmetry comes from the fact, that it can serialize data, but it cannot deserialize them. All you can do is to visit streams of data which might be a cheaper, especially if the data are not uniformly typed (if they were, a single preallocated object would do).

After extraction and working on it from time to time, recently I realized that maybe it'd be helpful for others to take a look at it. The span based serialization with the size estimator is something that I didn't found so far, or at least, something that was not explicitly used. If there's a similar serializer, just assume that great minds thinks alike ;-)

### Comments in the code

There are some comments in the code. At the same time I tried to make the code as clear as possible. Being given the fact, that I have a limited time though, I preferred to spend it on documenting principles and bits of protocol design in the readme rather than go through the code and insert lots of comments. But there are some in there :-)

### Future

Currently, this repository has no future. It has a past that has been squashed to a single commit, which should not influence readers capability to go through the code.

### Summary

I hope that you'll enjoy at least reading the principles behind the design. And do not hesitate to jump to the implementation. There's a lot of reflection, some pointers and loads of MSIL emitted in flight, but hey, it's all about being the fastest! Enjoy!
