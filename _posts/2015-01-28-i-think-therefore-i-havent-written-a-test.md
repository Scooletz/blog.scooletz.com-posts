---
layout: post
title: "I think therefore I haven't written a test"
date: 2015-01-28 12:00
author: scooletz
permalink: /2015/01/28/i-think-therefore-i-havent-written-a-test/
nocomments: true
categories: ["Personal"]
tags: ["personal", "professional"]
imported: true
---

It seems to me that veryfing one's thesis is one of the most important aspects of software engineer work. Quite often phrases started with 'it seems to me...' or 'think that...' are said in a way, that the listener takes for granted truthfulness of the sentence. Meanwhile it hasn't been proven or covered by a replicating test. I find it as a part of some kind of a social contract that 'if I say maybe and you partially confirm this then it's ok'. It isn't.

I'm not a TDD bigot, I use either TDD or tests themselves as a tool. But when a bisection is needed, when I have to search for a bug or simply verify my thesis, then the test provides a verifiable and repeatable way of answering a question with yes/no without all this hesitation. Just take a look at [protobuf-net cases](https://github.com/mgravell/protobuf-net/tree/master/SO11895998) named after StackOverflow questions.

I'm wishing you and myself truly binary answers with a high test coverage.
