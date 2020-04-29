---
layout: post
title: "Edge computing with Cloudflare Workers"
date: 2020-05-04 11:00
author: scooletz
permalink: /2020/05/04/edge-with-workers/
image: /img/2020/04/edge-workers.png
categories: ["serverless", "edge", "Cloudflare"]
tags: ["serverless", "edge", "Cloudflare"]
whitebackgroundimage: true
---

Recently I [migrated my blog](/2020/04/21/blog-refined/). I wanted to follow it up with some experiments that can be run on top of a statically generated page. One of them was a micro link shortener for [the tools I use](/tools). Another one, a bit more involving, was providing some kind of statistics for my blog. I wanted to do it without any additional scripts, keep it really simple and make it transparent for readers. To make this happen I used [Cloudflare Workers](https://workers.cloudflare.com) that delivers serverless, edge computation with an AP database and v8 isolates. I know it's a lot of mouthful words, so let me dissect it bit by bit.

The post is written it from my perspective, a cloud-aware person that wrote a serverless app or two.

### Basics

Cloudflare Workers are a serverless platform that is run on the edge. It means that your every worker is deployed

> to data centers across 200 cities in 90 countries to give it exceptional performance and reliability.

This means that it's different from regular `serverless` offering when you need to select a data center where your functions will be running. The interesting fact is that the same rule of distribution is used for both: the free tier and the paid one, so no matter what plan you choose, your workers will be run across the globe. The paid tier starts at 5$/month and is consumption based. The differences between the free and the paid plan are:

| Property |      Free      |  Paid |
|----------|----------------|------|
| Execution | first call is cold, then warmed up | always warmed-up |
| Database |    N/A   | AP, key-value store |
| Time budget | 10ms |    50ms |
| Executions | 100.000 / day | 10.000.000 / month + pay as you go |

The default choice of language is Javascript, but with WASM (Web Assembly), you can run almost anything. The rest of the article will focus on Javascript.

### Isolation and security

The isolation model and the security approach in Workers is quite interesting. Despite the traditional model of running a virtual machine or a docker image, Cloudflare engineers decided to use `isolates`. An isolate is a `Google Javascript v8` wrapped properly, so that the hosted worker cannot escape the sandbox. To prevent time-based attacks like Spectre etc. `Date.now()` is fixed for a single execution and you can't use it to measure branch prediction overhead. Cloudflare ensures that they measure and watch all the workers, so if a breach happened, they could easily provide a report. A single worker instance is limited to `128MB`. This includes global variables which you can use for caching between requests, but it does not cover the `CacheAPI` like cache, that is much bigger.

Additionally, `secrets` are available to pass `environmental variables` that should not be shown to the user.

Overall, I really like a fresh lightweight approach to isolation and security.

### API

If you're familiar with modern HTML API you will be familiar with the API that you can use in Workers. The following standards are available:

1. `CacheAPI` provided by a global `cache.default`.
1. `crypto` with the implementation of `SubtleCrypto`
1. `fetch`
1. `Promise` including gather/scatter with `Promise.all` and `async/await`.

The worker script is registered as a `fetch` interceptor. If it returns a response, then the original request is not passed. If it doesn't, the underlying service will be called. This is a great opportunity to provide static websites with API endpoints under the same domain, making them indistinguishable for the end user (that wants to reverse engineer your site). Let's take a look at a sample script from my stats app.

```javascript
addEventListener('fetch', event => {
    var request = event.request;
    var payload = {
        url: request.url,
        headers: headersToObject(request.headers)
    }

    // no event.respondWith, the original service will be called
    e.waitUntil(log(payload));
})
```

There are two interesting things to notice. The first, the lack of a call to `event.respondWith()`, meaning that the original service will be called. In my case this is a Jekyll-based static site. The other thing is a mysterious call to `e.waitUntil(Promise p)` that accepts a promise. This promise will be executed once the requestor is given the response and does not impact the response time. You can think of it as a great way to offload heavy tasks related with the database or other operations.

Beside a few specific APIs, you just write a `fetch` interceptor. How cool and simple is this!

### Limits

There are a few limits that you should take into consideration.

#### Time budget

Every worker is run within a tight time budget. For the Free Plan it's 10ms, for the Paid Plan it's 50ms. The good news is that it's not the wall clock timer (at least according to my measurements), so if you make a longer `fetch` request inside of your worker, it does not deplete the worker time budget.

#### Memory limit

The already mentioned memory limit of 128MB that you should not breach. If you breach it, your worker is killed. If you're close to it, GC might affect the speed of your worker instance. The outcome is that you may use global variables as a in-memory cache, but you should do it wisely.

#### Connection limit

The maximum number of concurrent connections is currently set to 6. This includes db calls and other `fetch` requests. It's ok to group promises with `Promise.all`, but again, keep the magic number in mind.

### Database

The database provided for Workers is accessible only in the Paid plan. It's a simple key-value store, falling into the `AP category` (from CAP theorem). This means that it's eventually consistent. According to the docs, it should be used as a read-mostly medium. Additionally, it provides no transactions and not conditional operations, so it's really a simple storage. The following APIs are available:

1. `put` - puts a value, with options like TTL
1. `delete` - deletes a key
1. `list` - lists keys
1. `list, prefix: p` - lists keys starting with p
1. `bulk` - bulk operations for adding and removing items

The interesting fact is that the database is accessible outside of the worker, so you can have another process that fills up the database for the workers. You can also use it for a `blue-green deployment` deploying the needed data before a new version is released.

For majority of cases you should treat the database as readonly, just to retrieve the data. The writes are heavily limited. Also, when designing workers remember that the data may be overwritten by other writes from other data centers.

### Routing

The routing is an interesting aspect of Workers. It's interesting because it's really limited, but with right usage of `empty routes` one can easily overcome limitations. The rules are as follows:

1. routes can be prefixed or suffixed with a `greedy * operator`
1. you can't use a full blown regular expression
1. a more specific route takes precedence over a less specific route
1. you can define empty routes, with no `Worker` being run for them

Now, consider the following scenario of my blog. I want to react to the whole meaningful navigation, but I want to skip fetches of images that are located in `/img` directory. Additionally, I'd like to omit assets located in `/assets`. The following set of routes would address it.

1. `https://blog.scooletz.com/img*` - empty
1. `https://blog.scooletz.com/assets*` - empty
1. `https://blog.scooletz.com/*` - MyWorker

With a proper exclusion/inclusion you can provide a set of routes that will quite likely cover all the requests that you want. Remember that you can filter them out on the Worker level.

### Summary

I must admit that being able to intercept any request for my site, without thinking where the code should be deployed is a game changer. It blurs the line between _just a static page_ and _a full blown web app_ providing and intriguing middle ground. This plus a geo-anognostic edge deployment of an app that is able to intercept _just what's needed_ is awesome and the provided API is more than enough to deliver a great platform for building app a `next generation edge hosted applications`. The performance of the site is not worse, as it was proxied by Cloudflare already. So far, I found no outstanding issues beside some glitches in UI which probably should not be used at all when working on serious applicationt that should be deployed via CI/CD pipeline.

The last but not least, a few resources to follow up:

1. [Workers home page](https://workers.cloudflare.com)
1. [Documentation](https://developers.cloudflare.com/workers/)
1. [Limitations](https://developers.cloudflare.com/workers/about/limits/)
1. Presentations:
    1. [Pick Your Region: Earth; Cloudflare Workers](https://www.infoq.com/presentations/cloudflare-workers/)
    1. [Real-World Examples of FaaS](https://www.infoq.com/presentations/faas-cloudflare-workers/)
