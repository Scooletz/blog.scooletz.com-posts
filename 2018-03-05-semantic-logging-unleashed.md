---
layout: post
title: "Semantic logging unleashed"
date: 2018-03-05 09:55
author: scooletz
permalink: /2018/03/05/semantic-logging-unleashed/
nocomments: true
image: /img/2018/02/stencil-default11.jpg
categories: ["Design"]
tags: ["logging", "semantic"]
imported: true
---

Who didn't use *printf* or *Console.WriteLine* to just something logged. Possibly, you were a bit more advanced and used a custom library that does this *line printing* to a separate file, or event a rolling file. What's printed is just a text. If you're aware enough, you'd put probably some 'around printed values'. In a moment of doubt, like a downtime or a serious client claiming that he lost money, you'd try to use these and the state of your app to find what's going on.

You may be on the other spectrum, using approaches like event sourcing where every single business decision is captured in a well-named, explicitly modeled object called an event. Your storage, providing a continuous log of all the events could be treated as the source of the truth. Is there anything in the middle of the spectrum?

Semantic logging is an approach where you model your log entries to be more explicit. You separate the template, from the actual values passed into the specific entry. See the following example:

``

The logging template, is constant. It informs about an error that occurred. In this case, it's a failed log. Is there a schema? Of course, there is. It's not strict, as it may change whenever a developer augments the statement. Nevertheless, the first part:

``

provides a schema. The value, passed as the second parameter, is the value for the occurrence of the event. Depending on the storage system, it can be indexed and searched for. Same for templates. If you have this part specifically separated (and we do have it in the semantic approach), it's easy to index against this part and search for all *entries of failed user logons*.

I'm not a fan of implicitness, I'd say I'm totally on the opposite side of the spectrum, still, I find the semantic logging approach, explicit enough to capture important information, without thinking about all the schema celebration. Eventually, the only consumer of the schema and the payload is the logging tool. If it's smart enough to make sense out of it, without spinning tens of VMs or using tens of GB of RAM, I'm fine with this semi-strict approach.
