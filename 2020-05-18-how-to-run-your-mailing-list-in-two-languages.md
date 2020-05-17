---
layout: post
title: "How to run your mailing list in two languages and not get crazy?"
date: 2020-05-18 11:00
author: scooletz
permalink: /2020/05/18/how-to-run-your-mailing-list
image: /img/2020/05/mailing-list.png
categories: ["mailing", "ConvertKit"]
tags: ["mailing", "ConvertKit"]
whitebackgroundimage: true
---

If English is not your mother tongue and you use it for blogging like I do, you probably thought about switching between English and your native one. The whole process gets even more complex when you have a mailing list! Should the list be bilingual or maybe one should stick to just English? Or native one?

In this post I'll describe my approach, and how I use awesome features from [ConvertKit](https://scooletz.com/links/convertkit) to send emails that match my readers preferences.

### Inputs and choices

If you subscribe to a mailing list and author speaks your language, it would be great if I could explicitly state my interest in receiving emails in this language. Moreover, it would be awesome if I could do this in any time because I might have subscribed to it on a page, that was written in English, as this makes the input not so reliable in making the assumption about the preferred language. How could this be done?

To make this happen, it needs to be placed in a place that is added to every email and that is somewhat correlated with setting options. The best place that I found is a `footer`.

![mailing footer](/img/2020/05/mailing-footer.png)

The receiver can easily click `Polish` to add a `PL` tag or remove it by clicking `English`. Tags in ConvertKits are nothing more than attributes that are assigned to a subscriber, like hashtags in social media, etc. In my case, the presence of `PL` tag means that the subscriber speaks Polish. If it's not there, I assume English. If you want to read more about custom footers in ConvertKit [this article](https://help.convertkit.com/en/articles/2502535-custom-unsubscribe-links) will help you.

It looks like keeping subscribers' preferences is quite simple. There's one more concern that should be addressed.

### Changing your language in flight

The concern that was bugging me for a while was related to changing the preferred language in flight of a sequence of emails. Mailing tools often provide a way to write a sequence of emails up front and share knowledge with subscribers according to a schedule that is separate for each subscriber. In case of [ConvertKit](https://scooletz.com/links/convertkit), the sequences are called... `Sequences` and provide you with a lot of tooling, that can be used to address this concern. Let's take a look at a few emails from my sequence behind my Domain Driven Desing ebook, [Master of Aggregates](https://masterofaggregates.com).

![mails in alternating languages](/img/2020/05/mailing-alternate.png)

As you can see above:

1. emails are written within one `sequence`
1. the same email is written once in English, once in Polish

Keeping same things together makes a lot of sense! If I want to change the content, I can easily edit both versions in one place. This approach also addresses one more issue.

I can imagine a person that subscribed to an English version of the sequence and then changed their preferred language to Polish. Should they receive the same sequence in Polish and be bored to death by repetition? Not at all! As ConvertKit tracks the fact of sending a sequence to a given subscriber, with this approach, the same person will never be sent the same email sequence.

With this in mind, a subscriber can easily change their language in flight and enjoy emails being sent in their language. They will just use the footer mentioned above.

### The right language of your emails

There's one thing that is bugging me though. What about the sequence. How does it know which email should be sent to which subscriber? If I had a drum roll sample I'd play it now, as this is an awesome feature for ConvertKit sequences.

![filtering emails in sequences](/img/2020/05/mailing-filter.png)

As you can see above, every single email in a sequence can be filtered out on the basis of some criteria. In my case I exclude:

1. Polish emails for subscribers who prefer English
1. English emails for subscribers who prefer Polish

How cool is that?

### Summary

I hope that this post shows you how to not get crazy when keeping your list bilingual. Maybe the same ideas can be mapped to other mailing tools, but I'll stick to [ConvertKit](https://scooletz.com/links/convertkit). Doing this kind of heavy lifting in there is really easy.
