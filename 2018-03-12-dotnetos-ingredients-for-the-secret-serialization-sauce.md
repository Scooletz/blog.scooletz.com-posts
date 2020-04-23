---
layout: post
title: "Dotnetos - ingredients for The Secret Serialization Sauce"
date: 2018-03-12 09:55
author: scooletz
permalink: /2018/03/12/dotnetos-ingredients-for-the-secret-serialization-sauce/
image: /img/2018/03/Dotnetos.png
categories: ["presentations", "Dotnetos"]
tags: ["presentations", "Dotnetos"]
whitebackgroundimage: true
nocomments: true
---

Today is the first day of our first [Dotnetos](http://dotnetos.org) tour across Poland. I thought that it would be useful to gather all the links, repositories in one post and share it with you in a nicely wrapped burrito preserving the spiciness of the The Secret Serialization Sauce. All right: here goes links:

1. XML - sorry, no links for this
1. [Jil](https://github.com/kevin-montrose/Jil) - the fastest JSON serializer for .NET, seriously, it's crazy fast. If you're interested why is it that fast [this pearl from its design](http://blog.scooletz.com/2018/02/05/pearls-jil-primitive-serialization/) can reveal some ideas behind.
1. [protobuf-net](https://github.com/mgravell/protobuf-net) - my favorite implementation of Google standard for protocol buffers, for .NET. In the recent version it supports [discriminated unions](http://blog.scooletz.com/2018/01/25/pearls-the-protobufs-discriminated-union/) and much more. Yummy!
1. [Hyperion](https://github.com/akkadotnet/Hyperion)/Wire - a new player in the serialization area. Started as a Wire, later on forked by Akka.NET team. I invested some time to [make it faster](http://blog.scooletz.com/2016/08/09/wire-improvements/).
1. [Simple Binary Encoding](https://github.com/real-logic/simple-binary-encoding) - we're getting soooo close to the wire. It's simple, binary and uses a special field ordering to allow reading values of a constant length with ease.
1. [NServiceBus monitoring](http://go.particular.net/monitor) - it uses a custom, extremely efficient protocol for reporting all metrics from a monitored endpoint to the monitoring instance. All the optimizations, the [whole approach behind it is described in here](http://blog.scooletz.com/2018/02/26/pearls-the-protocol-for-monitoring-nservicebus/).
1. New wave o serializers - my prediction is that within a few months, a year maybe, we'll see a new wave of serializers, or at least a wave of refurbished once, based on Span and Memory (read this [in-depth article](http://adamsitnik.com/Span/) by Adam Sitnik for more info). I'm working on something related to this area and results are truly amazing.

Here you have! All the needed ingredients for The Secret Serialization Sauce. Enjoy 🌶🌶🌶
