---
layout: post
title: "Idempotence, pt. 2"
date: 2011-10-24 11:00
author: scooletz
permalink: /2011/10/24/idempotence-pt-2/
nocomments: true
categories: ["Architecture", "DDD", "Design"]
tags: ["architecture", "design", "idempotent"]
imported: true
---

In the previous post a few operations were taken into consideration, whether there are (not) idempotent. For the sake of reference, here there are:

* Marked as default
* Money transfer '500$' ordered to 'x' account
* Label 'leave sth for the future month' added

If we consider 'idempotent' as an operation which can be applied multiple times *in a row*, then all the operations *overriding* previous values of some properties are idempotent. Having some entity marked as default 5 times does not change the fact that it is default. That's for sure. What about provisioning 'x' account with 500$? Can this type of operation can be reapplied multiple times? Of course not, because it does not override any property, it *changes the state*, by interacting with a previous one. The same goes for 'labeling', of course if there is no compensation introduced (select only unique labels before saving, which would allow reapplying).

What if you want your system to be resistant to operations resend multiple times? The simplest solution is to add unique identifier for each operation and storing them is a lookup (hashtable). Each time the operation arrives, the lookup is checked whether there this operation was already processed. If so, skip it.

There is one additional condition is to have the lookup transactional with a storage you save the states. This condition is a simple 'all-or-none' for storing the result of operation with the fact, that this specific operation was already applied. Otherwise, if lookup would be updated in the first place and storing the state after the operation failed, there would be no change saved. The same applies to a situation, where the lookup is updated at the very end. The operation result is saved, adding info about operation to lookup fails and the next time the same operation arrives it is applied one more time. Having that said, lookup must be transactional with the medium where state is saved.
