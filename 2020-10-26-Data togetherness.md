---
layout: post
title: "Data togetherness"
date: 2020-10-26 11:00
author: scooletz
permalink: /2020/10/26/data-togetherness
image: /img/2020/10/together.png
categories: ["serverless", "design", "performance"]
tags: ["serverless", "design", "performance"]
whitebackgroundimage: true
---

There are moments when a design and architecture of software speaks  through code. It might be the simplicity of a specific piece or  nightmarish gymnastics one has to do to make something out of it. Recently I was working on an architecture of one solution. I was going  back and forth trying to cut the cost of performing some calculations. It stroke me that there was a quality of code that was a  result of higher level assumption. It was something related to bringing computation to data and splitting services the right way. The best term that I came up with though, was **data togetherness**.

### Fetch'em all

Consider the following code

```csharp
var user = await db.GetAsync<User>(id);
var product = await db.GetAsync<Product>(order.ProductId);
var price = await ruleEngine.CalculatePrice(product, user);

PerformAction(product, price, user.Address);
```

As an opinionated group of professionals we could have discussions about:

- separating responsibilities
- distilling the one and only core domain
- splitting services properly
- the love-hate relationship with `async-await`

etc. The impact on this small piece of code though, could be described as a lot of things to do before it performs its action. Once everything what's needed is fetched, the real action can proceed.

### Data togetherness

The fact of having everything what's needed makes it so much simpler to work with. We're back in the land of pure computation, where good old-fashioned algorithms can be applied. We have all the needed data together in one place, hence the data togetherness. Now, one could ask whether it's just:

1. a pleasing property of code,
1. a need of writing pure functions,
1. a sloppiness in using new techniques like asynchronous programming?

I'd say that none of the mentioned above is as important in the light of computation as commodity (read: serverless, edge) as the fact that a computation can be performed in an effective way. Having all the needed data ready to be processed, is simple, effective and scalable at the same time. It's not always achievable. At the same time, it's a really pleasant property that you can observe and take into consideration when designing a modern system.

### The protocol case

I forgot about the initial case, the protocol described in the initial paragraph. The initial draft included sending a message to an endpoint with some metadata. The metadata were some identifiers from a kind of helper dictionaries, that later on were used in the processing. This approach made the send operation really cheap as it included:

1. a bit of computation (some hashing),
1. registering dictionary entries (background job)
1. sending a message,

The receiver, when processing the message, would fetch the needed data from dictionaries and process it. This removed the **data togetherness**, requiring additional work. It had an additional drawback. In the initial design it was the receiver, who was responsible for fetching the data, potentially caching it to make it faster. As always with caching, it was hard to forsee what and when will be used. Moving this bit of computation to the message sender, just by moving a piece of optimized code to the client, enabled doing it in the best possible way, by keeping the data together when needed.

### Summary

I don't treat **data togetherness** as a ground-breaking architectural principle, or the only law to follow when writing and designing software. I'd rather sum it up as an interesting factor to observe and take into consideration when moving into the new world of the computation as commodity.