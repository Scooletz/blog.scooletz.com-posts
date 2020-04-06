---
layout: post
title: "Protobuf-net: message versioning"
date: 2012-12-17 10:00
author: scooletz
permalink: /2012/12/17/protobuf-net-message-versioning/
nocomments: true
categories: ["Protobuf-net"]
tags: ["message versioning"]
imported: true
---

Welcome again. Recently I'm involved in a project where [Protobuf-net](http://code.google.com/p/protobuf-net/) library is used for the message serialization concern. The Protobuf-net library was developed by Marc Gravell of Stackoverflow. Beside standard Google Protocol Buffers concepts there is a plenty of .NET based options, like handling derivation, using standard Data Contracts, etc. This is the first post of a few, which will deal with some aspects of Protobuf-net which might be nontrivial for a beginner. Beside the posts, a project [Protopedia](https://github.com/Scooletz/Protopedia) was created to show cases of Protobuf usages. All the examples below are stored in this repository. All the posts requires a reader to get accustomed to [the official Google Protocol Buffers documentation](https://developers.google.com/protocol-buffers/).

### Versioning

The versioning of interfaces is a standard computer science problem. How to deal with sth which was released as looking in one way and after internal changes of the system it must be changed publicly.
Consider following scenario of having two versions of one message located [here](https://github.com/Scooletz/Protopedia/blob/master/Protopedia/Versioning/Messages.cs). As you can see, there are a few changes like the change of the name, the addition and removal of some fields. Imagine that a service A runs on the old version of the message. A sevice B uses a new version. Is it possible to send this message from A to B, and then back to A and have all the data stored in the message? With no loosing anything appended by the B service? With Protobuf's Extensible class used as a base class it's possible. The only thing one should remember is to do not reuse ProtoMemberAttribute tags' values of field removed in the past. New fields should always be added with new tags' values.

### How does it work?

When Protobufs deserialize a message, all the data with tags found in the message contract are deserialized into the specific fields, the rest of data is hold in a private 'storage' of the Extensible class. During serialization, these additional fields are added to the binary form allowing another message consumer to retrieve fields according o their version of the message.
