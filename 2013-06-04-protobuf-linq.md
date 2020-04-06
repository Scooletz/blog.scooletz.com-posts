---
layout: post
title: "Protobuf-linq"
date: 2013-06-04 10:00
author: scooletz
permalink: /2013/06/04/protobuf-linq/
nocomments: true
categories: ["Protobuf-net"]
tags: ["OSS", "projections", "serialization"]
imported: true
---

I had an idea about querying and projecting over big streams of messages serialized with Google Protocol Buffers. If one needs only a few fields to his/her projection, why don't make it implicit and prepare an optimal way of deserializing only these fields? That's the way [Protobuf-linq](https://github.com/Scooletz/protobuf-linq "Protobuf-linq") has been born. It's simple, fast and eager to help you iterate over big streams of data.
Check it out!
