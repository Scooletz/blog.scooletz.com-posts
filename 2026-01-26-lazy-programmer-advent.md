---
layout: post
title: "The advent of a lazy software engineer"
date: 2026-01-26 06:00
author: scooletz
permalink: /2026/01/26/lazy-programmer-advent
image: /img/2026/lazy-programmer-advent.webp
categories: ["LLM", "agents", "coding"]
tags: ["LLM", "agents", "coding"]
whitebackgroundimage: true
twitter: false
---

This post is not about LLMs and your new coding assistants that you spawn at will. It’s about something that has been prevalent for much longer than that, and due to this fact and Lindy's law, I think it will be much harder to delegate.  
We all heard about the new laziness that crawls up on us. It is all about spawning as many coding agents as possible, often swarming them up, enjoying the ride of being “just a reviewer”. Even though it’s kind of lazy, I don’t think it’s lazy enough.

## What is this new laziness like?

We live in a truly new era. Every time I have a few tasks being worked on by Codex in the cloud, I am truly in awe. So far, my PR (personal record, not the pull request\!) was juggling a maximum of 6 at the same time, asking for minor or major adjustments. It still sometimes feels like a micromanagement, do this, do that, don't you dare to xyz, but still. One person can choreograph so many things nowadays\!

And don’t get me started with fancy tooling like MCPs or skill files\! The sole fact that there are agents out there doing this work for you. How marvelous it is. But it also brings the fundamental question of any knowledge worker: 

> Are you working on the most important thing? 

Now, this can be extended to:

> Are your agents working on the most important thing?

Should they burn through tokens to build up things, almost from scratch? Or could we use their token budgets in a better way?

## It’s about…

The whole point about coding agents being able to code things… Now, we said it. Is it about **coding things** or maybe, it’s about **building things**?

Here, I just said it. Building. Not coding a piece of software, not drafting a database schema, not making schema management easier to maintain. It’s about **building value**. And one thing that is required to build, is **gluing pieces together**.

Now, let’s count how many times you wrote a code that was taking some data from a database and pushed it elsewhere. Or performed slight alterations and stored them back. Or accepted an influx of data to store them in a database. It’s ok, if you’re getting paid for the gluing, but are you? Is this repeatable work, that can be LLMed away nowadays, worth spending your time on? The maintenance of it will be a cost anyway.

I’d ask further, is it worth being LLMed away over and over again? I don’t think this is true,  especially if we consider that **something has to run it**\! No matter how serverless your solution is, no matter how well scalable your K8 pods are, if you need to run it, you need to run it. And it’s better not to have this obligation. So…

## It’s time to delegate

I think the times where a piece of infrastructure can stand on its own are over. The ability to delegate a piece of the gluing inward, to a part of the solution that you buy, not build, is a must have. I think that you and your coding agents should become terribly lazy. You should push and delegate not only the coding part to the agents, not only the gluing to the infrastructure provided by 3rd parties, but even further. What if the infrastructure can perform some work on your behalf?

And let’s reiterate again. If you spend your CPU cycles on running a serverless function or “just a docker image” somewhere to react to every single event, and you don’t delegate things to the infrastructure, I think it’s time to re-evaluate the infrastructure and your choices.

Examples?

### Azure Event Grid

Are you thinking about reacting to blobs being created or deleted? Do you need to scan for changes (like it's the 20th century or something)? You can be notified of things happening via Azure Event Grid. Blob Storage has [its own set of events](https://learn.microsoft.com/en-us/azure/event-grid/event-schema-blob-storage%20) ready to be raised for you. You need to consume them, this is for sure, but the publishing part is done for you\!

### DynamoDB Streams

Let’s assume that you build on AWS DynamoDB and you want to react to some data changes. You configure an AWS Lambda function to be triggered by the stream of changes. The platform itself becomes the glue.

### RavenDB ETL for Kafka

Some documents from RavenDB should have their parts streamed through to Kafka? Extract an identifier plus some data. The gluing is there provided as a [Kafka ETL task](https://docs.ravendb.net/7.1/studio/database/tasks/ongoing-tasks/kafka-etl-task). Even in cloud instances. You delegate it. You don’t build and maintain it.

## The box

I think every single solution that is built could use some help from a box. No matter who builds it, they need to become lazy to use a part of a solution that is capable of understanding a bit more and reaching a bit further. Something that is capable of publishing information, reaching out for data, and then, providing the gluing on its own. It should be able to store some procedures to be performed on behalf of the user who configured it (or an agent, if this is the case) as well. It’s building time, not gluing time. Leave the gluing and the expert knowledge required to make it work for the box and enjoy the full E2E experience. Out of the box\!
