---
layout: post
title: "RampUp journey ends"
date: 2016-06-27 09:00
author: scooletz
permalink: /2016/06/27/rampup-journey-ends/
nocomments: true
categories: ["Open Source", "RampUp"]
tags: ["OSS", "RampUpNet"]
imported: true
---

[My RampUp project](https://github.com/Scooletz/RampUp) has been an important journey to me. I verified my knowledge about modern hardware and dealt with interesting cases writing a really low level code in .NET/C#. Finally, writing code that does not allocate is demanding but doable. This journey ends today.

RampUp has been greatly influenced by [Aeron](https://github.com/real-logic/Aeron), a tool for publishing your messages really fast. You can easily saturate the bandwidth and get very low service time. Fortunately, for .NET ecosystem, Adaptive Consulting has just opened an Aeron client port [here](https://github.com/AdaptiveConsulting/Aeron.NET). This is unfortunate for RampUp as this port covers ~70-80% of RampUp implementation. Going on with RampUp would mean, that I need to chase Aeron and compete somehow with Aeron.NET to get an .NET environment attention, and in result, participation (PRs. issues, actual use cases). This is what I don't want to do.

RampUp source code won't be removed as still, I find it interesting and it shows some really interesting low level cases of CLR. I hope you enjoyed the ride. Time to move on :)
