---
layout: post
title: "Cacheable REST API for time series"
date: 2014-09-29 11:00
author: scooletz
permalink: /2014/09/29/cacheable-rest-api-for-time-series/
nocomments: true
categories: []
tags: ["API", "Cacheability", "REST"]
imported: true
---

I's been a while since last blog post and mapping back the blog domain. Let's restart with something fency: REST API!

### The problem

Lets look at any time series gathered in a eventually consistent medium (with a given time threshold). The entries/events gathered from various inputs are stored in a persistent data store. Queries for data are nothing than requests for given time slice. How one could design a REST API for accessing such a store?

### The solution

Let's start with the first request for a sample [timeseries.scooletz.com ](http://timeseries.scooletz.com)link. To read it, in an ASP MVC Web API routing way would be something:

<span style="color:#000000;">GET http://timeseries.scooletz.com/api/data</span>

In the result I'd return a list of entries from *now - t1* seconds till *now - t2* seconds. *t1* and *t2* are selected arbitrary to match the eventual nature of the store and process of data gathering. Of course t1 > t2.

How to provide a navigation for it? Nothing simpler comes to my mind than putting [LINK header ](http://www.w3.org/wiki/LinkHeader)which lets you provide navigation in a semantic way. The proposed value would be:

Link: <<span style="color:#000000;">http://timeseries.scooletz.com/api/data</span>/before/2014_09_29_23_59_10>; rel="prev"

This would allow to get previous to the current head page. Why the second part is a multiplication of 10? That's because of the time-chunk size. I've chosen to group all the entries in a 10 second chunk. This also means that the first request can contain from no data to full chunk span. There is an additional reason revealed later.

The second page, beside data placed in the body of the response would have the following value of the link header:

Link: <<span style="color:#000000;">http://timeseries.scooletz.com/api/data</span>/**before**/2014_09_29_23_59_00>; rel="prev", <<span style="color:#000000;">http://timeseries.scooletz.com/api/data</span>/**after**/2014_09_29_23_59_10>; rel="next"

The third page, and all the following ones would contain the following structure of the link header:

Link: <<span style="color:#000000;">http://timeseries.scooletz.com/api/data</span>/**before**/2014_09_29_23_58_50>; rel="prev", <<span style="color:#000000;">http://timeseries.scooletz.com/api/data</span>/**before**/2014_09_29_23_59_10>; rel="next"

As you can see, from the third page on, all the pages uses **before** links for navigation. There is a very good reason for it. The never expiring http cache headers set for the data in the past, which does not change.The second page could be cached, but the **after** link would always be responded with a non cacheable response. As the time passes, the response for the *<span style="color:#000000;">http://timeseries.scooletz.com/api/data</span>/before/2014_09_29_23_59_10* would change their headers to

Link: <<span style="color:#000000;">http://timeseries.scooletz.com/api/data</span>/**before**/2014_09_29_23_59_00>; rel="prev", <<span style="color:#000000;">http://timeseries.scooletz.com/api/data</span>/**before**/2014_09_29_23_59_20>; rel="next"

These principles can be sum up to:

1. cache-never the root request for data
1. cache-forever **before** responses as they point to the consistent past

1. cache-never **after** reponses as they point to the data being stillgathered

Which results in only non-cacheable first request, if the client moves to the past (using prev links). One can find it very helpful with a paradigm of [infinite scroll](http://www.infinite-scroll.com/). Only data for a few first entries would be fetched, all the latter would go from the browser/client cache.

### Etags

You should remeber about ETags as well. Firefox, for instance, when a user hits F5, issues [a request with max-age=0 ](https://bugzilla.mozilla.org/show_bug.cgi?id=419194)trashing caches. If you add an ETag equal to the date included in the before link, you can verify it on the server side and immediately respond with 304 NotModified. The before links contains immutable data after all :)
