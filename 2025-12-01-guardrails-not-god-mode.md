---
layout: post
title: "AI Agents Need Guardrails, Not God-Mode MCP Access"
date: 2025-12-01 06:00
author: scooletz
permalink: /2025/12/01/guardrails-not-god-mode
image: /img/2025/guardrails-not-god-mode.webp
categories: ["ai", "agentic", "database"]
tags: ["ai", "agentic", "database"]
whitebackgroundimage: true
twitter: false
---

*Just slam an MCP on top of it*. This is the mantra that we all hear much too often nowadays. And even if we follow to the letter the spec provided via the official Model Context Protocol page, without guardrails, we can land ourselves in serious trouble.

# A tale of client, server, and this third one

Even though everyone talks about MCPs, it's worth exploring what are the actual components needed to satisfy the `Protocol` in `MCP`. First, there are two sides: the client and the server. You may think of a client as a connection to a server. The client then, is used by a host. This means that a single host app can use multiple clients to connect to their servers. What do they provide though?

A server provides: prompts, resources, and tools. 

**Prompts** are templates that provide you with the required inputs and result in a template being filled in. 

**Resources** are addressable chunks of data (think of anything that has URI). You can `get`, `list`, and `subscribe` to resources. On top of that, they provide you with `templates` so that a server can tell you how to query for any resource. An example of a template could be `reports://{year}/{month}` where if the `year` and `month` are filled properly, you can access the underlying monthly report.

**Tools** are the most common MCP capability that is used. Similar to prompts, they define inputs and outputs and allow performing actual actions on the server, by performing execution on the server side when triggered. This is the part that is referenced and implemented in various MCP flavours out there.

The client side, beside defining **roots** that point to a directory, allows the server to ask the clients for favors. Either **sampling** that asks the client to complete the task using LLM, or **elicitation**, they allow to perform a kind of callback on the client end.

But mostly, it's about the tools. And this is where the problem resides.

# You can do everything, but ask first

Let's consider a case of a database that is quite permissive regarding what agents can do. What do I mean by being permissive here? After all, we can limit the data access to be read only. This should greatly reduce the surface of a potential error, right? For simple cases, provided for demo purposes, this clearly can be true. What about more complicated ones?

Let's consider a case where the data that a particular user (with its agents) can access should be strongly limited. What could be such cases?

1. Sales - you should not be allowed to access and steal leads from other sales engineers. You and your agents should only look at your data.  
1. HR & payroll - as an employee, you should be able to look and reason about only your PTOs, payslips, sick leaves. Assuming you're a bad actor, should your agent be able to look at coworkers data and help you to come up with the maximum number of sick leaves still smaller than the other team members?  
3. Health - as a patient, accessing your lab results and discussing their preliminary meaning with a chat bot, would you be happy to share it with other patients and their agents? 

We can create countless examples like this. They will share one thing in common that is not quite liked by the demoware. They are bound to some context. It could be something as simple as `userId` or maybe a `unitId`. Still having an ability to express an unbreachable contract of the business context in which the functionality resides is a *must-have* for a system.

# The unbreachable contract of the business context

As shown above, all the hand waving and asking nicely won't do if a context can be breached. How would you approach a construction of an unreachable one then?

First, it would require accepting some parameters. Already mentioned `userId` or `employeeId` or anything else that is used for scoping.

```csharp
var userId = session.UserId;
var parameters = new { UserId = userId};
```

Then, when a conversation is started, it should allow passing the initial parameters, so that the conversation is aware of them.

```csharp
myAi.StartConversation (parameters)
```

The last prerequisite is ability to define data access in a way that is parameter-aware. You need some good old-fashioned SQL-like params for that.

```sql
from Deals d where d.SalesEngineer = $UserId
```

Combining these three: defining parameters, passing them to a conversation, defining queries with parameters in mind, allows you what we wanted from the beginning:

**If there’s a business need** to narrow down the data to a specific business context of a given conversation, **you should be able to express it**. 

One database that uses this approach is RavenDB with its AI capabilities (see: [agents](https://docs.ravendb.net/7.1/ai-integration/ai-agents/ai-agents_start)). Just let me show you the following snippet adheres to the rules above:

```csharp
  var agent = await store.AI.CreateAgentAsync(agent, new Performer
  {
     suggestedReward = "your suggestions for a reward",
     employeeId = "the ID of the employee that made the largest profit",
     profit = "the profit the employee made"
  });

   // Set chat ID, prefix, agent parameters.
   var chat = store.AI.Conversation(
     agent.Identifier,
     "Performers/",
     new AiConversationCreationOptions().AddParameter("country", "France"));
```

What you get is a conversation that is bounded by the parameter it received. With queries implemented in a parameter-aware way, you just got yourself out of a lot of legal questions.

# The worst answer

The worst answer to a question "will it breach" that you may provide is "yes, if you ask nicely". Don't fool yourself and your users. If you want to enter the agent-land, do it in a way that allows you to express the business requirements fully. So that the agent, if asked nicely or being threatened somehow won't have the ability to bail out itself with the data they should not access at all.  
