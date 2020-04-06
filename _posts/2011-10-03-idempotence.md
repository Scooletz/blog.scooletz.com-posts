---
layout: post
title: "Idempotence"
date: 2011-10-03 20:00
author: scooletz
permalink: /2011/10/03/idempotence/
nocomments: true
categories: ["DDD"]
tags: ["CQRS", "idempotent"]
imported: true
---

Recently, I've been reading a lot of [dddcrqs](http://groups.google.com/group/dddcqrs) group. The reason was a project I'm writing, which will be implemented in a DDD paradigm using CQRS and the event sourcing. As I read about about them, I found over one year old [Greg's post](http://codebetter.com/gregyoung/2010/08/12/idempotency-vs-distibuted-transactions/) considering pros and cons of *idempotence*. The definition of idempotence is pretty straight forward. We call an operation idempotent iff:
*f(f(x)) = f(x)*
which simply means, that applying the same function multiple times brings the same result. In mathematics, you can find a constant function, giving the same result for each argument, hence, during multiple application. That's the mathematician point of view. What about computer science?
It can be easily brought to the DDD field, especially, the event-sourced implementation of it. For those who don't know what the event sourcing is, take a look [here](http://martinfowler.com/eaaDev/EventSourcing.html). Consider the following chain of events for a client's bank account:

* Marked as default
* Money transfer '500$' ordered to 'x' account
* Label 'leave sth for the future month' added

Which of them would you consider idempotent and under which conditions?
