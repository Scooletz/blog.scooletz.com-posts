---
layout: post
title: "Web API caching get wrong"
date: 2014-06-05 11:00
author: scooletz
permalink: /2014/06/05/web-api-caching-get-wrong/
nocomments: true
categories: ["Http"]
tags: ["http", "REST", "web API"]
imported: true
---

I read/watch a lot of stuff published on the [infoq](http://www.infoq.com/) site. I enjoy it in the majority of cases and find it valid. Recently I read an article about [Web APIs and Select N+1 problem](http://www.infoq.com/articles/N-Plus-1) and it lacks the very basic information one should provide when writing about the web and http performance.
The post discusses structuring your Web API and providing links/identifiers to other resources one should query to get the full information. It's easy to imagine that returning a collection of identifiers, for example ids of the books belonging to the given category can bring more requests to your server. A client querying over books will hit your app one by one performing from a load test to a fully developed DOS. The answer to this question is given in following points:

* Denormalize and build read models
* Parallelising calls
* Using Async patterns
* Optimising threading model and network throttles
* ...

### What is missing is

the basic http mechanism provided by the specification: cache headers and ETags. There's no mention about properly tagging your responses to allow return 304 if the client asks for data that didn't change. The http caching, its expiration are not mentioned as well. Recently Greg Young posted a great article about [leveraging http caching](http://codebetter.com/gregyoung/2013/05/28/why-cant-i-update-an-event/). The best quote summing the whole take on it from Greg's article would be:

> This is often a hard lesson to learn for developers. More often than not you should not try to scale your own software but instead prefer to scale commoditized things. Building performant and scalable things is hard, the smaller the surface area the better. Which is a more complex problem a basic reverse proxy or your business domain?

Before getting into fancy caching systems, understand your responses, cache forever what isn't changing and ETag with version things that may change. Then, when you have a performance issue turn into more complex solutions.

### UPDATE:

For sake of reference, the author of the Infoq post reponded to my tweet in [here](https://twitter.com/Scooletz/status/474476955060805633).
