---
layout: post
title: "ProtoDescriptor"
date: 2014-01-13 11:00
author: scooletz
permalink: /2014/01/13/protodescriptor/
nocomments: true
categories: ["Protobuf-net"]
tags: []
imported: true
---

Recently I've created (with some porting from another project) a simple library which allows parsing .proto files and storing them in a model. The library offers serialization/deserialization of the mentioned model. I hope I ship dynamic genaration of protobuf-net classes as well. It would allow creation of self-desriptive streams (contract added at the very beginning of a file) discoverable via reflection (you got class, you IEnumerable of this class' object) and queryable. It has some potential in it.

[https://github.com/Scooletz/ProtoDescriptor](https://github.com/Scooletz/ProtoDescriptor)
