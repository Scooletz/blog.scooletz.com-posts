---
layout: post
title: "You're using wrong numbers"
date: 2017-05-25 08:55
author: scooletz
permalink: /2017/05/25/youre-using-wrong-numbers/
nocomments: true
image: /img/2017/05/stocksnap_07cc9ba522.jpg
categories: ["Personal"]
tags: ["docs", "documenting", "writing"]
imported: true
---

### TL;DR

When testing, we often use constants to show that a number used in two places has the same meaning, not only value. How can we apply this when writing documentation or giving examples of algorithms you want to use?

### Meaningless enough

Consider the first example

> 2 - 1 = 1
1 - 0 = 1
1 + 1 = 2

Can you guess which "ones" were added at the last line? They are results of the first two equations. Or maybe not? Let's take a look at the slightly altered example

> 12342 - 1 = 12341
1 - 33333 = 33332
1 + 1 = 2

Can you now see which ones were used? It's obvious.

### Number as identifiers

When working with numbers you may try to use them as identifiers. When a value is needed only for a local calculation make it *meaningless enough*. A reader won't spend much time on it. They will read and remember one-digit numbers and apply them in following lines

### Summary

Writing documentation isn't an easy task. Writing good documentation is even harder. When dealing with algorithms, making numbers unique can improve the overall understanding of samples you provide.
