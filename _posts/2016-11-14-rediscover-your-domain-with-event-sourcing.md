---
layout: post
title: "Rediscover your domain with Event Sourcing"
date: 2016-11-14 09:50
author: scooletz
permalink: /2016/11/14/rediscover-your-domain-with-event-sourcing/
nocomments: true
image: /img/2016/11/stocksnap_h07qq5zx7s.jpg
categories: ["DDD", "Event sourcing"]
tags: ["domain modelling", "event sourcing"]
imported: true
---

### **TL;DR**

Beside common advantages of event sourcing like the auditing, projections and sticking closely to the domain, you can use events to discover the domain again and provide meaningful insights to your business.

### **Include.Metadata**

I've already described the idea of [enriching your events](http://blog.scooletz.com/2015/08/11/enriching-your-events-with-important-metadata/). This is the main enabler for analyzing your events in various way. The basic metadata one could are:

* date
* action performer
* on behalf of who action is taken

You could add a screen of your app and IP address and many many more.

### **Reason**

Having these additional data, it's quite easy to aggregate all the events of a specific user. This, with time attached, could provide various information:

* how the work is distributed during a work day
* how big is the area of business handled by a single user
* is the user behavior pattern the same all the time or maybe somebody has overtaken this account?

The same with projection by event type:

* is it a frequent business event
* are these event clustered in time - maybe two events are only one event

Or looking at a mixed projection finding sequences of events for a user that might indicate:

* an opportunity for remodeling your implementation
* finding hot spots in the application.

### **Rediscover**

![stocksnap_crlx1ud87t](/img/2016/11/stocksnap_crlx1ud87t.jpg)

All of the above may be treated as simple aggregations/projections. On the other hand, they may provide important trends for a system and might be used to get an event based insight to the business domain. Can you imagine the business being informed about a high probability of a successful cross selling of two or three products? That's where a competitive advantage can be born.
