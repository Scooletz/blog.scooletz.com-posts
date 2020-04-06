---
layout: post
title: "The tests that come to your mind, when all you have are messages"
date: 2015-10-16 11:00
author: scooletz
permalink: /2015/10/16/the-tests-that-come-to-your-mind-when-all-you-have-are-messages/
categories: ["Testing"]
tags: ["integration", "tests", "unit tests"]
imported: true
---

The majority of the developers do the enterprise related development. Probably, you're one of them. Even if pet projects or OSS contributions are important to you, enterprise related development is the thing you spend with majority of your professional time. Enterprise applications are quite dependent. You don't have just a database and an app sitting on top of it. There are many services, applications, databases that your solution need to talk to. Additionally, the communication may be very different, from REST services using http to TCP, queues, or even shared databases... The question which arises is are the true unit tests the tests that you should use to cover your whole application? How do you test at all an application which is in the worst case just a mediator?

How would you describe the API provided by your service, the database, queues etc? Follow my way of thinking and consider them as output-input. In the center there's your application configured with a fancy DI container, properly configured. Consider now calling an API to get a current account state. Your application access the database to get cached glossaries to provide properly named properties, gets some data from external transactional service, possibly sends a notification with some queue. Now, let turn these into named entities:

* GetAccountsData (call to your API)

    *   FetchGlossaries (call to your DB)

    *   GetAccounts (call to transactional system)

    *   NotifyCompliance (push to queue)

How would you test this flow? Using strict mocks, some substitutions? I'd say, instead of interfaces, services repositories go with a simple approach of messages and handlers. The whole flow is a result of initial message GetAccountsData. Having provided responses to FetchGlossaries, GetAccounts, NotifyCompliance in a simple form: just take the request object and compare it serialized version with the one registered as fake. If that fits, return the preregistered response. At the end you should assert the result of GetAccountsData.

To minimize the set of faked responses, fail the test if the response was not used at all. Fail if it was used to many times if you want. You may go into world of mocking libraries, but it's not needed when every call is a simple single object passed to a mediator, dispatching it to a handler. You know the thing about objects. You easily deep compare them with a simple serialization output.

Is it a test? Yes. Is it a unit test? Not at all. You're asserting the whole configured system. Is it worth? In my opinion with that approach you can have more business oriented tests, describing some scenarios rather than tests handling external mock/assert libraries. If I would this approach everywhere? Probably not. With a highly collaborative and dependent applications it seems to me know as a time and sanity saver. Define input-output and assert the system as a whole. Nothing more, nothing less.
