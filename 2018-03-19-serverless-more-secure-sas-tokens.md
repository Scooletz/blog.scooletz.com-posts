---
layout: post
title: "Serverless & calling no functions at all"
date: 2018-03-19 09:55
author: scooletz
permalink: /2018/03/19/serverless-more-secure-sas-tokens/
nocomments: true
image: /img/2018/03/stencil-default6.jpg
categories: ["Azure", "Serverless"]
tags: ["azure", "security"]
imported: true
---

If you ever used serverless approach, you know that limiting the number of executions can save your money. This is true especially, when you run your app under a consumption plan. Once the free budget is breached, you'll start paying for every execution and consumption of precious GigaByte-seconds. Is there a way to minimize this cost?

### A cheaper option

One of the approaches that is often overlooked by people starting in serverless environment is not calling a function at all. If the only thing that your functions does is obtaining a blob from the storage, or getting a single entity from a table, there is a cheaper option. The option is to use SAS tokens, which stands for *Shared Access Signatures Token.* Let's take a look at the following example, that generates an URL that allows you to access blobs, list them and read them from a specific storage account.

https://gist.github.com/Scooletz/5b3d55ef5e5bc7857b3d23bde22dc629

Shared Access Signature is a signed token that enables a bearer to perform specific actions. For instance, you can encode into an url values enabling user to get a specific blob or to add messages to an Azure Storage Queue. The bearer will be able only to perform the set of actions that was specified when the url was created (you can find more, about SAS tokens in [Azure docs](https://docs.microsoft.com/en-us/azure/storage/common/storage-dotnet-shared-access-signature-part-1)). How can we use this to lower our costs?

Instead of returning the payload of a blob, a function can return a properly scoped token, that enables a set of operations. In this way, it will be the client's responsibility to call Azure services directly, without going through the function proxy. It may not only lower your serverless bill, but also, decrease the latency, as the value is not copied first to the function and then sent to a client, but it's used directly, with no proxy at all.

The idea above looks great, but there's a single if. What if the token is stolen and another party uses it?

### Time and address

The first option to address a possibility for leakage is to use time-based tokens. Instead of issuing an infinite token, issue a token for a specific time and refresh it from the client side, before the previous token expires. Another option is to use another feature of SAS tokens. A majority of methods for obtaining them, enables to pass [IPAddressOrRange](https://docs.microsoft.com/en-us/dotnet/api/microsoft.windowsazure.storage.ipaddressorrange?view=azure-dotnet). With this, you can specify that the token will be correct only if the caller of a specific operation, meets specified criteria. If you issue a token for a single IP, then you can greatly limit the surface of a potential attack.

### Summary

A good, old saying that *doing nothing is cheaper than doing anything* applies also to serverless. It might not only cheaper to not call a function, but also much faster as there's no additional step, just for copying the data. It's about time to renew your SAS token, so you'd better hurry up!
