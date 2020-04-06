---
layout: post
title: "Pricing SaaS in the clouds"
date: 2017-11-20 09:55
author: scooletz
permalink: /2017/11/20/pricing-saas-in-the-clouds/
nocomments: true
image: /img/2017/11/stencil-default2.jpg
categories: ["Azure", "Serverless"]
tags: ["azure", "FaaS"]
imported: true
---

*Why is it so pricey?* This is a question that might have popped in your head too many times. Especially, when looking at the pricing pages of SaaS applications. The second that might have followed up, is *Why? What's behind this pricing model*? *How did they come up with it?* Of course one answer could be *I don't care. They did it to get rich. With my money!*, but this isn't very constructive, is it? What would be the minimum price you'd charge for a single user, or a single account of your app? These are much better questions to ask.

Recently, I've been playing with Azure Functions. They provide this beautiful FaaS (Function as a Service) environment, where you pay for what you use. In a Consumption Plan, you don't even pay for you app lying in there as long as nobody uses. Not paying for having no users is a good thing. Having users and paying something is a much better situation though. Imagine now, that you have your first account registered. Let's put aside the cost of staff/work/development. How much money do you need to handle this single account. How would you estimate costs?

I think that using word *estimation* in this case, would be a really underestimation. It's so easy to put a single Function App, with a single storage account and just run a synthetic workload. A single account for one month. Then, using Azure cost management, just to take a look at your bill. See the numbers. No guessing, no estimation, but real costs, real money. Now, with these numbers, you can go back to the pricing model and put something on top of it, just to make it work for you. And for clouds' sake, remember to [make it rain](http://blog.scooletz.com/2017/10/30/heavy-cloud-but-no-rain/)!
