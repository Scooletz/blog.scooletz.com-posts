---
layout: post
title: "Enriching your events with important metadata"
date: 2015-08-11 11:00
author: scooletz
permalink: /2015/08/11/enriching-your-events-with-important-metadata/
categories: ["DDD", "Event sourcing"]
tags: ["EDA", "event driven architecture", "event sourcing"]
imported: true
---

When considering the application of [event sourcing ](https://msdn.microsoft.com/en-us/library/jj591559.aspx)it's quite common to allow a common part for all the events, the metadata. Various stores handle it in separate but common ways. [EventStore](https://geteventstore.com/) lets you append the metadata with events. The same you can do with [NEventStore](https://github.com/NEventStore) using headers. But what information can be useful to store in the metadata, which info is worth to store despite the fact that it was not captured in the creation of the model?

### Audit data

The most common case considered by various lists and blog posts are audit data. This set of data can be described as:

1. who? - simply store the user id of the action invoker
1. when? - the timestamp of the action and the event(s)
1. why? - the serialized intent/action of the actor

That's an obvious choice and you can easily find examples filled with repartitioning by the **who** or gather event in a time frame or window as it's done in the complex event processing. But is there something more one could store? Is it there any particular set of additional dimensions that are worth to remember?

### Important metadata

The event sourcing deals with the effect of the actions. An action executed on a state results in an action according to the current implementation. Wait. The current implementation? Yes, the implementation of your aggregate can change and it will either because of bug fixing or introducing new features. Wouldn't it be nice if the version, like a commit id (SHA1 for gitters) or a semantic version could be stored with the event as well? Imagine that you published a broken version and your business sold 100 tickets before fixing a bug. It'd be nice to be able which events were created on the basis of the broken implementation. Having this knowledge you can easily compensate transactions performed by the broken implementation.

It's quite common to introduce canary releases, feature toggling and A/B tests for users. With automated deployment and small code enhancement all of the mentioned approaches are feasible to have on a project board. If you consider the toggles or different implementation coexisting in the very same moment, storing the version only may be not enough. How about adding information which features were applied for the action? Just create a simple set of features enabled, or map feature-status and add it to the event as well. Having this and the command, it's easy to repeat the process. Additionally, it's easy to result in your A/B experiments. Just run the scan for events with A enabled and another for the B ones.

### Optimization (when needed)

If you think that this is too much, create a lookup for sets of *versions x features.* It's not that big and is repeatable across many users, hence you can easily optimize storing the set elsewhere, under a reference key. You can serialize this map and calculate SHA1, put the values in a map (a table will do as well) and use identifiers to put them in the event. There's plenty of options to shift the load either to the query (lookups) or to the storage (store everything as named metadata).

### Summing up

If you create an event sourced architecture, consider adding the temporal dimension (version) and a bit of configuration to the metadata. Once you have it, it's much easier to reason about the sources of your events and introduce tooling like compensation. There's no such thing like too much data, is there?
