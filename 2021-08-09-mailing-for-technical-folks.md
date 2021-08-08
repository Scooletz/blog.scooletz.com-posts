---
layout: post
title: "Mailing with ConvertKit for technical folks"
date: 2021-08-09 11:00
author: scooletz
permalink: /2021/08/09/mailing-for-technical-folks
image: /img/2020/09/mail.png
categories: ["mailing", "ConvertKit"]
tags: ["mailing", "ConvertKit"]
whitebackgroundimage: true
---

The topic of running a mailing list is still hot. I already covered it in a few entries from various points of view. One hasn't been touched yet and as I was asked the same question recently, let me flush out why a technical folk should consider [ConvertKit](https://scooletz.com/links/convertkit) as a good option.

### Jekyll/Liquid Templates

The **static page generation** is the new black. This blog as many others use this approach to have the output html generated and hosted in a cheap/free environment like GitHub Pages. There are many generators nowadays, including [Jekyll](https://jekyllrb.com/), that is used natively by the mentioned GitHub pages, [Hugo](https://gohugo.io) and many more. It' interesting, that whenever you write an email in ConvertKit, either as a part of a sequence or a single time broadcast, it allows you to **customize it** using Liquid syntax, the one that you know from Jekyll. You can see all the options at [Advanced Email Personalization with Liquid](https://help.convertkit.com/en/articles/2502501-advanced-email-personalization-with-liquid).

The most important part for me was that knowing Liquid from Jekyll, I was able to transfer this knowledge to the mailing platform. I don't write these snippets on daily basis, but having in mind that **it's just Liquid** makes it a lot easier.

### Webhooks, API and integrations

The majority of the things you can do from UI, are accessible from ConvertKit's API that you can learn about from the [developers' page](https://developers.convertkit.com/). Beside API, ConvertKit also provides webhooks. Let's be honest. This is isn't something groundbreaking and any service is expected to provide that. It's worth to mention though that if you prefer to use tools like [Integromat](https://scooletz.com/links/integromat) or other low-code tools, there are many integration scenarios supported by these platforms.

It's worth to mention that if you use third-party tools, they often come with the ConvertKit integrations on their own, so it's easy to transfer your subscribers to the mailing platform of your choice.

### Technical support

The last but not least is the technical support. I admit that I used it a several cases and my experience was always positive.

The first case that comes to my mind is related to the automation that I configured and had a problem to reason about. The support provided me with an image that clearly showed the path where it went wrong.

The second was related to my idea of using segments in the conditional content. This is not possible as segments are groups of users and a subscriber variable that is passed down, does not have it provided. Of course, there's a way to address it by storing whatever needed in a custom field of a subscriber and then again, use the Liquid templates. Again, it was not about the technical issue, but rather about the positive way and the speed of answering my question. This was top notch.

The third was related to a change which we did for the Dotnetos Platform to support the shopping of [Dotnetos online courses](https://dotnetos.org/products/). Instead of using ConvertKit Products we changed the behavior to use tags instead. Again, the support helped with some operational aspects of it.

### Summary

I hope that this entry shows some technical aspects of ConvertKit and why, after a few years, I'm still using this platform. I hope that if you consider starting a mailing list, [ConvertKit](https://scooletz.com/links/convertkit) will become your tool. It delivers what it promises and it does it in a really good way.
