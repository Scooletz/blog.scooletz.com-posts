---
layout: post
title: "How I built and run my e-shop for 0.7$/month using Azure Functions and a few more"
date: 2019-09-02 08:55
author: scooletz
permalink: /2019/09/02/how-i-built-and-run-my-e-shop-for-0-07-month-using-azure-functions-and-a-few-more/
image: /img/2019/09/m2.png
categories: ["Azure", "Azure Functions", "serverless"]
tags: ["Azure", "Azure Functions", "serverless"]
whitebackgroundimage: true
nocomments: true
---

This post summarizes my work in building a small online shop for providing [my Polish course for Master of Aggregates.](https://masterofaggregates.pl).

## Not invented here syndrome

The first that might come to your head is that it's 2019 and it's not the best time to build a custom shop. Let me describe what were requirements first, what was the offer that I found on the market and how I compared them. I'll start with listing some components that I needed to have connected to make it work and end it with answering the question, why on Earth did I build it myself.

### ConvertKit

I've tried three email marketing tools and so far [ConvertKit is my favorite one](https://scooletz.com/links/convertkit). It enables a lot and has a purchase module that is simple and sufficient enough for me to register that somebody has ordered a product. I know, you can use tags, separate lists or whatever. I wanted to have it separate and keep it simple at the same time.

It's worth to mention that it has a well-described web API that enables to send emails to a specific person (you can start a sequence of emails for a single person).

### Payment provider

There are a lot of payments providers, some of them, they are not accessible for businesses conducted on Polish grounds. The one that I used was Poland-ready and had an API providing integration that is done by building a link on the your server side. The link is secured by generating a MAC (Message Authentication Code) which enables verifying both the data integrity and the authentication of a message. This means that the payment links can be generated without usage of the external service. This is how this provider worked.

### Academy

The finalized order required to get access to the course site. This required sending an invitation link. Before sending it, the link should be generated with an API call.

### Modelling orders

An order was an interesting thing to model. It had to capture the whole intent of the order (in case of a any failure to enable communication with the ordering person). A separate problem was how to act on the payment success. The payment provider requires a timely response, containing a specific text. As the number of integrated components is not small and consists of:

1. updating the order status
1. obtaining an invitation link
1. registering a purchase in [ConvertKit](https://scooletz.com/links/convertkit)
1. sending an initial mail sequence

These for sure should not be done as a response. How this could be done then?

### Azure functions

As will be revealed in a second, there's not that much to do on the server side! There will be a single action of registering the order. Then, a single action to react to the payment. Then, a few more to do the rest.

## The Answer

![moa-arch.jpg](/img/2019/09/moa-arch.png)The shop itself has been published as a static page. With Jekyll, and *_data* files it's easy to have a simple *for* loop that generates your shop. Additionally, when the order has always a single item, it's easy to generate the order page as well. The final step is registering the order which is done by an Azure function.

The Azure Functions in a consumption plan has a problem of a cold start. Whenever you function is not executed for a long time, it will be de-provisioned and the very first request will take some time to get it running. The easiest way to address it was to add a tracing request. Whenever somebody enters the shop, a background request is sent to ordering function to make it ready for obtaining the request. With this approach, when a customer is done with entering all the data, the function should be already running.

As the generation of the payment link is fast and can be done in memory, this could be treated as a part of creating an order. You want to create an order? Give me data, I'll generate a link and this will be your order aggregate.

The next step is redirecting to the payment gateway side and waiting for the response. The response is sent via web request, which hits another function. This is a special function though, that only accepts that the payment is confirmed and stores it in a queue. Nothing more! Why is that? To keep the processing which can be really lengthy from the acknowledgement that should be done really fast. Now the shop have the acknowledgement stored and can process it.

The processing, is done in background and contains of several steps I mentioned before: creating the link, sending an email, marking order as completed, etc. You could use several strategies to model and develop it. The most important assumption in this case was to: make the processing bullet-proof and ready to be retried whenever a part of the process fails. In this case, the poison-queue for Azure Functions was a savor. I had two integration problems (third parties responding in a strange way) and being able to retry the process, was a great thing to have.

This leads to the end of the process, where the customer:

1. has an invitation link in their inbox (sent by ConvertKit sequence API with a custom field)
1. has a purchase registered, both in ConvertKit and the ordering system
1. finally, can access the platform.

## All the whys

**Why did I do it myself?** After looking at different market options, there was nothing that was matching all my needs. Maybe I didn't look long enough, but in every case there was some integration part to be written. It's worth to mention that existing solutions  were not able to use full functionality of other parties (again, Purchases in ConvertKit). This meant that I'd either have written 60-80% of it, or that I'd have written it as a whole. As I had previous experiences with all the technologies, Jekyll, Azure Functions, ConvertKit and still needed to provide some integration with the course page, I chose to spent a few hours writing a few Azure functions instead of spending a few hours... wait a sec! :P

**How does it cost only 0.7$/month?** I didn't process millions of orders :-) That's the first part of the answer. It's run in the consumption plan, a.k.a Pay as You Go. The functions are used only for API! The whole page, including checkout is static so there's no need for running function to support it. It wasn't a whole month of actual load (the initial 3 weeks was a load from my test app).

**So where's the app?** It's somewhere between. Or maybe there's no app, and there's just a business solution solving a problem of delivering a content for users that bought a course.

## Summary

I hope this posts was interesting and showed you some perspective on running cheap, integration-like piece of software. At the same time, even beside being just a a few functions, modelling the core of it was crucial, to make it resistant to all the failures that might occur.
