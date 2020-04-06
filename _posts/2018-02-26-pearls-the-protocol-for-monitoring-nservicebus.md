---
layout: post
title: "Pearls: the protocol for monitoring NServiceBus"
date: 2018-02-26 09:55
author: scooletz
permalink: /2018/02/26/pearls-the-protocol-for-monitoring-nservicebus/
nocomments: true
image: /img/2018/02/stencil-default4.jpg
categories: ["Design"]
tags: ["design", "NServiceBus", "pearls"]
imported: true
---

It's time for another pearl of design, speed and beauty at the same time. Today, I'm bringing you a protocol used by NServiceBus to efficiently report its measurements to a monitoring endpoint. It's really cool. Take [a look](https://docs.particular.net/monitoring/metrics/in-servicepulse)! Not that I co-authored it or something... ;-)

### Measure everything

One of the assumptions behind monitoring NServiceBus was ability to measure everything. By everything, I mean a few values like Processing Time, Criticial Time, etc. This, multiplied by a number of messages an endpoint processes can easily add up. Of course, if your endpoints processes 1 or 2 messages per second, the way you serialize data won't make a difference. Now, imagine processing 1000 messages a sec. How would you record and report 1000 messages every single second? Do you think that "just use JSON" would work in this case? Nope, it would not.

### How to report

NServiceBus is all about messages, and being given, that the messaging infrastructure is already in place, using messages for reporting messaging performance was the easiest choice. Yes, it gets a bit meta (sending messages about messages) but this was also the easiest to use for clients. As I mentioned, everything was already in place.

### Protocol

As you can imagine, a custom protocol for custom needs like this could help. There were several items that needed to be sent for every item being reported:

1. the reporting time
1. the value of a metric (depending on a metrics type it can have a different meaning)
1. the message type

This triple, enables a lot of aggregations and enables dealing with out of order messages (temporal ordering) if needed. How to report these three values. Let's consider first a dummy approach and estimate needed sizes:

1. the reporting time - DateTime (8 bytes)
1. the value of a metric - long (8 bytes)
1. the message type (**N bytes using UTF8 encoding**)

You can see that beside 16 bytes, we're paying a huge tax for sending the message type over and over again. Sending it 1000 times a second does not make sense, does it? What we could do is to send every message type once per message and assign an identifier to reuse it in a single message. This would prefix every message with a dictionary of message types used in the specific message *Dictionary<string,int>* and leave the tuple in the following shape:

1. the reporting time - DateTime (8 bytes)
1. the value of a metric - long (8 bytes)
1. the message **type id** - int (4 bytes)

20 bytes for a single measurement is not a big number. Can we do better? You bet!

As measurements are done in a temporal proximity, the difference between reporting times won't be that big. If we extracted the minimal date to the header, we could just send difference between the starting date and a date for the entry. This would make the tuple look like:

1. the reporting **time difference** - int (4 bytes)

1. the value of a metric - long (8 bytes)
1. the message type - int (4 bytes)

16 bytes per measurement? Even if we're recording 1000 messages a sec this gives just 16kb. It's not that big.

### **Final protocol**

The final protocol consists of:

1. the prefix

    1.  the minimum date for all the entries in a message (8 bytes)

    2.  the dictionary of message types mapped to ints (variable length)
1. the array of

    1.  tuples each having

    1.  the reporting time difference - int (4 bytes)

    2.  the value of a metric - long (8 bytes)

    3.  the message type - int (4 bytes)


With these schema being written binary, we can measure everything.

### Summary

Writing measurements is fun. One part, not mentioned here, is doing it in a thread/task friendly manner. The other, even better, is to design protocols that can deal with the flood of values, and won't break because someone pushed a pedal to the metal.
