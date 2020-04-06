---
layout: post
title: "Event sourcing and failure handling"
date: 2014-05-23 11:00
author: scooletz
permalink: /2014/05/23/event-sourcing-and-failure-handling/
nocomments: true
categories: ["Event sourcing"]
tags: ["ES", "EventSourcing"]
imported: true
---

Currently I'm workingwith a project using event sourcing as its primary source of truth and the log in the same time (a standard advantage). There are some commands, which may throw an exception if the given condition is not satisfied. The exception propagates to the service and after transformation is displayed to the user. The fact of throwing the exception is not marked as an event. From the point of consistency it's good: an event isn't appended, the state does not change, when an exception occurs. What is lost is a notion of failure.
A simple proposal is to think a bit more before throwing an exception and ending a command with nothing changed. One may append a ThisCriticalCommandFailedEvent with nothing but the standard event headers (like time, user performing command, etc.) or something with a better name and return a result equal to the exception thrown. The event can be used later, when you want to analyze failures of executing commands.
