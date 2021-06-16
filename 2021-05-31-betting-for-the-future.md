---
layout: post
title: "Betting for the Future"
date: 2021-05-31 11:00
author: scooletz
permalink: /2021/05/31/betting-for-the-future
image: /img/2021/05/future.png
categories: ["personal", "business"]
tags: ["personal", "business"]
whitebackgroundimage: true
---

_The future is unknown! This is terra incognita! You cannot foresee things!_ and a few other statements are often used when describing one's attempts to  place some bets for the future. As a person, who spent a big part of their life in IT industry, it's tempting for me to place some bets though. How would the future look? What is already on the table? Can we forsee the future? Let me pick my top 3 candidates.

### Edge computing and the Ultimate Commodity Game

The ultimate goal of your code is to **get it executed** and deliver a business value. The environment, where the execution is performed may seem like a relevant thing. But it's not. Currently we observe that the execution **environment is getting commoditized**. It's done in a non aggressive manner, by providing serverless execution environments. Moving the execution to the commodity area is not only about _I don't care how it's run, just run it_. It's also about how fast, with what latency it can be run.

The edge computing, meaning, running the computation close to the user, is the next step for the serverless. Your app is run not only **without you caring** where it is run. It's also run as close as possible to the end user, without they asking why does it run that slow!

### Computational Integrity and the Great Offloading

Nowadays the **computation and its verification** is run in one place. A function or a service hosts some code in a form of a library or an executable and runs it whenever requested. If we assume that the code was deployed properly (no supply chain attacks) the execution will provide the **same result for the same inputs**. It's as simple as that: the execution is the verification of itself. Now consider the following questions:

1. What if one could split these two though?
1. What if someone could separate running a heavy computation from verifying whether the run was performed correctly?
1. What if the verification was much cheaper than the actual execution?

Sounds intriguing? I bet it does! One of the application of this approaches that is already happening is related to Ethereum and its scalability. I'm talking about a family of protocols that includes SNARKs, STARKs and other approaches for splitting the verification from the execution. This is not a holy grail of all the computation, but rather **a great leverage** that can be used in environments with an **asymmetrical execution costs**. Think about places where a bit more costly execution with much cheaper verification could be used... Looking for a good example? Think: **blockchain**.

In block chain every single transaction, every single action is executed and validated by every participant in the consensus quorum. With the same function and the same inputs, the result must be the same. This creates a great overhead because all the nodes need to run the very same computation! This creates a **linear amplification of the cost of execution**! If the split was applied, and nodes would be responsible for executing only the verification part, then there can be a single actor performing a full execution that is later on verified on-chain with a so much lower cost of it. How powerful is that!

### NoOps and the Great Overseer

No input or an error lost. Everything is a data. And all your apps, services and functions are observed. This sounds a bit like _1984_, but it's a better version of it.

We're slowly getting there but we're still not there. Still, when an app or a service is created there are some long debates how to log things, how to observe things, how to react to things. I think that within a few years this will be addressed, making more things **measurable** and much easier to digest. If this is combined with the movement to the commodity in the execution area, we can land in a place where all things are monitored and where there's always enough of data to reason about a situation and whether it's cheaper to just leave it as it is, or maybe, one should rewrite this component where the money are being leaked through some improper usage of resources.

This possibly will be followed by **general marketplaces**. After all, if you're good with writing a function that transforms an image, why don't you earn something with providing it as a... function for general use. Think of it as **Dyson's spheres**  but in the realm of computer components, both hardware (it needs to be hosted somewhere after all) and software (using other functions as a service).

This is already happening with various tools in various platforms, but my perception is that it's **just the beginning**.
