---
layout: post
title: "Code kata with Business Rules"
date: 2016-12-08 09:55
author: scooletz
permalink: /2016/12/08/code-kata-with-business-rules/
image: /img/2016/11/stocksnap_avmron1nhs.jpg
categories: ["Architecture", "DDD", "Design"]
tags: ["architecture", "design"]
imported: true
---

### TL;DR

How many times you were given an implicit requirements that you'd create one application or two services? How many times the architecture and design were predetermined before any modelling with business stake holders? Let's take a dive into a code kata, that will reveal much more than code.

### Kata

The kata we'll be working on is presented [here](http://codekata.com/kata/kata16-business-rules/). It covers writing a tool for a set of business rules gathered across the whole company. The business rules depends on the payment (the fact that it is done) and some other conditions, for example:

* *If the payment is for a physical product, generate a packing slip for shipping.*
* *If the payment is for a book, create a duplicate packing slip for the royalty department.*
* *If the payment is for a membership, activate that membership.*

The starting point for every rule is a payment that is accepted. Another observation is that these rules are scattered across the whole company (as the author mentions *Carol on the second floor handles that kind of order*). Do you think that having a new single application that gathers all the rules is the way to go?

### Contexts

If a mythical Carol is responsible for some part of the rules, maybe another department/team is responsible for membership? What about the payments? Is the bookstore part of your organization really interested if a payment was done with a credit card or a transfer? Is a membership rule really valid outside of the membership context? Is it needed to anyone without membership specific knowledge should be able to say when the membership is activated?

I hope you see the way through these questions. There are multiple contexts that are somehow dependent but that are not a truly one:

* payments - a part responsible for accepting money and making them transferred to the company's account
* membership - taking care of (possibly) accounts, monitoring activity, activating/disactivating accounts
* bookstore/videostore or simply store - the sales part
* shipping - for physical products

Are these areas connected? Of course they are!

### **PaymentReceived**

The first visible connector is the payment. To be precise, the fact of receiving it, which can be described in a passive tense *PaymentReceived*. You can imagine, when requesting a membership, a payment is required. This can be perceived as a whole process, but could be split into following phases:

* gathering membership data
* requesting a payment
* receiving a payment
* completing the membership order

This is the Membership point of view. As you can see it requests a payment but does not handle it. We will see in the next post how it can be solved.
