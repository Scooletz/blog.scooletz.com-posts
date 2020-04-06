---
layout: post
title: "On breaking the monolith"
date: 2018-06-04 08:55
author: scooletz
permalink: /2018/06/04/on-breaking-the-monolith/
nocomments: true
image: /img/2018/05/breaking.jpg
categories: ["Architecture", "Design"]
tags: ["architecture", "design"]
imported: true
---

### TL;DR

Are you dreaming about breaking a monolithic application? Some of us do. What I'd prefer to hear is **breaking a monolithic application into pieces that fit**. This would be much appropriate and much more advantageous than just breaking this monolith, wouldn't it? We want it to work, after all. Let's go through different approaches and perspectives one can use to work with the mighty monolith. Maybe it's not that bad after all.

### Just leave it (TM)

If you ever wanted to chop your monolith into pieces, this urge might have gone wrong. Especially, when there was no reason to chop it. Sometimes it works, and its parts won't be used by other elements. Maybe it's an app that does not require scaling. Maybe it's worth to leave it as [The Majestic Monolith](https://m.signalvnoise.com/the-majestic-monolith-29166d022228).

### Evolution, not revolution

Have you ever heard or been a part of a big rewrite? How many succeeded? Probably none. I can tell a story or two as well. The revolutionary approach fails so often, not only in real life, but also in IT. If you dream about being the one and delivering another Big Bang, please don't. It's highly likely that you'll fail. There's another way to make big changes. A much simpler and safer way.

### Chopping, piece by piece

You won't win war in one day. You'll need many days and many battles. Winning battles will make you stronger in waging them and keep moving forward. Choose a small item first, and try to separate it from from the monolith. How to do it? In a non breaking manner. How to do it, and what can you do? Read ahead.

### Adding bits to allow more

Imagine any application, that provides some API (either for SPA frontend or other apps in the system). Now, let's assume that one of the methods, deep, behind all the shiny REST API looks like this

```csharp
void RegisterComplaint(int clientId, string complaint)
{
db.Save(new Complaint (clientId, complaint);
}
```

The very first step, would be adding some reactivity, by adding events, even by storing them

```csharp
void RegisterComplaint(int clientId, string complaint)
{
// transaction applied auto-magically ;-)
var id = db.Save(new Complaint (clientId, complaint);
db.Save(new ComplaintRegistered(id));
}
```

We could also use a messaging in here. The fact is that we just enhanced the existing process, without changing it.

### Wait or react

The result of introducing this event is simple. Now, as a caller, we can either wait for the result and, for instance, query the service for additional complaint data or we can react to the event. By doing this small change we enabled other parts of the app to work with the complaint module in different ways. Some of them might call it in the old fashion, some of them can use the new approach.

### Keep moving forward

The change described above might not be finished at all. The mentioned part might be left in the monolith with just a bit of make up, playing a good citizen in the reactive/message based system. It depends on the module and on your judgement.

### Strangling leftovers

With this evolutionary approach we can refactor the complaint module into more and more reactive way. Finally, if you want to, you may remove the API and use messaging, or leave API whenever synchronous is required. With this approach, and spreading the non-synchronous approach, you can move evolutionary through your whole app, to make it, eventually, an ecosystem of services (you might call them #microservices or turn them into #serverless to get a few hype points :wink:).

### Observe, add, remove, repeat

Observe what you have on the table, add what's needed in a non-breaking fashion then remove leftovers and repeat. An iterative, simple way of making a change. Yes, it won't be this easy in every system and yes, it might be hard. But with every small change you'll learn and with every small victory you'll know how to change the very next piece.

### You know the pieces fit

I hope that you'll know that pieces fit, as you'll be the one who separated them. But don't worry, doing it piece by piece, the big picture will be not lost all. It will become more.
