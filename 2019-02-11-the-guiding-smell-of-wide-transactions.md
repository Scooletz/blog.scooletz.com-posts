---
layout: post
title: "The guiding smell of wide transactions"
date: 2019-02-11 09:55
author: scooletz
permalink: /2019/02/11/the-guiding-smell-of-wide-transactions/
image: /img/2019/02/tx.jpg
categories: ["DDD"]
tags: ["domain driven design"]
imported: true
---

If you can feel a bad smell in the air, you can tell that something is rotten. The very same rule applies to sniffing out transactions spanning several entities to make a business operation complete. You may call these entities aggregates, you may call them Foo or Bar, but if transactions are wide, you're in trouble.

### But this is how it is

Top one excuse for a bed smell of wide transactions is:

> *But this is how my business works*

And nope, it does not. If you own a bakery, and somebody comes to buy a loaf of bread won't be going through all the baskets, touching every single loaf to select the one. You'll just get one.

If you own a travel agency and somebody buys a trip, you won't be calling the hotel, the airlines, the weather man to get everything aligned. You'll just happily accept the payment, you will assign the number to this deal and make everything work underneath. If there's a problem with a plane, you'll rent another one etc.

### There's one body holding the invariant

The rule for making it work is to select a single body responsible for validating and holding the invariant behind business operation.

Somebody wants to buy something? Just register the order and deal with the lack of the product later.

You want to transfer money from the account? Just confirm that the source account has enough money and register the transaction. What if there's no destination account? You can move money back to the source account. The existence of the destination account is not needed to request a transfer!

> Want to know more about modelling aggregates? Grab a free e-book at [masterofaggregates.com](https://masterofaggregates.com)

### But what if

Maybe your problem is really unique. Maybe you are unlucky and you need to span this wide transaction. There's a huge chance though, that you can model it out. Simple as this.
