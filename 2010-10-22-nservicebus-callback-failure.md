---
layout: post
title: "NServiceBus callback failure"
date: 2010-10-22 20:00
author: scooletz
permalink: /2010/10/22/nservicebus-callback-failure/
categories: ["Architecture", "NServiceBus"]
tags: ["architecture"]
imported: true
---

Today I found a bug in NServiceBus, which occured just before  publishing our project. I isolated the bug, and published the unit test  on the SourceForge, but I'm not quite sure whether the project is still  alive (there are a few tickets untouched). The link to the isolated  case: [http://sourceforge.net/tracker/?func=detail&atid=1009068&aid=3092090&group_id=209277](http://sourceforge.net/tracker/?func=detail&atid=1009068&aid=3092090&group_id=209277)

The problem is a race condition which can occur when the service using  Reply, replies before the callback is registered to the message. The  scenario is simple: the reply message correlation id is not found in the  dictionary and the message is consumed with no trace left. The callback  registration occurs after this and hence, it will never be fired at  all. Sad but true: this eliminates NServiceBus' synchronous way to go.

It seems that for now, I cannot use callbacks at all (replying with  messages handled by handlers works perfectly since it does not use this  'correlation stuff'). Worth to notice, that I've got to redo a lot of work because of this 'feature'.

Any thoughts about that?
