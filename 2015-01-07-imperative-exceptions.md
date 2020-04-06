---
layout: post
title: "Imperative exceptions"
date: 2015-01-07 11:00
author: scooletz
permalink: /2015/01/07/imperative-exceptions/
categories: ["Design"]
tags: ["API", "design", "exceptions", "libraries"]
imported: true
---

If you develop software in .NET, you probably use exceptions. Or at least, handle them, even by simply logging. Beside providing an easy way to deal with runtime errors, exceptions are frequently met during the initial phase of using a library or a framework, when you don't know API yet and try to do something the other improper way. Consider one of the exceptions: [KeyNotFoundException](http://msdn.microsoft.com/en-us/library/system.collections.generic.keynotfoundexception%28v=vs.110%29.aspx). It's thrown by a dictionary when your program tries to get the key which hasn't been added. The question what should you do when you encounter this error.
The truth is that the message of this exception isn't descriptive enough. It simply states that:

> System.Collections.Generic.KeyNotFoundException:
> The given key was not present in the dictionary.

This doesn't provide you any meaningful information. After getting this exception, you still don't know what key was missing. I'd prefer to get the missing key, event as a string representation. Later on, when the exception is logged, one can tell what was missing. But that's only a prelude.
What about cases when you receive the meaningful and well-described exception like:

> You haven't registered any handler for this event

Does it help you to solve this problem? If you know the library and you met this exception before, it'll be easy to fix. What if it's your first encounter? Then, providing an imperative part like:

> You haven't registered any handler for RoomBooked event. Register handler using bus.Register(hander)

is extremely helpful and lets a developer to maintain the focus on the code rather than switching to searching through StackOverflow.
