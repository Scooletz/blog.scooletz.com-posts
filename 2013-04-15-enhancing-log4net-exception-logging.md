---
layout: post
title: "Enhancing log4net exception logging"
date: 2013-04-15 10:00
author: scooletz
permalink: /2013/04/15/enhancing-log4net-exception-logging/
nocomments: true
categories: ["C#", "log4net"]
tags: ["event log", "EventId", "EventLogAppender"]
imported: true
---

Using a library like log4net is surely helpful in your day to day app development. One of the most important cases where an entry should be logged is an application exception occurring on the production environment. It's quite common to use Windows' event logs as a storage for log entries and log4net has a special appender which is used in this cases called simply EventLogAppender. All it does by default is appending an entry to the Application event log using a parametrized event source name. It does not set an event id or a category of the entry, disabling a simple querying of entries by this param. There are imperative ways of adding one, like by adding an element EventID or Category to a dictionary *log4net.ThreadContext.Properties*. This makes you either wrap each call to log4net with a proxy (it's useful and may be used in your project) or pollute your code with plenty of a logging logic. Can it be done in another way?

### Filter to the rescue

It's not quite hard to provide a better, separated way of enhancing your log entries on the information based on the exception object being thrown. One of them is using a filter, which has an ability to access an exception by a *LoggingEvent* instance. A simple code of doing it can be found in the following gist.

https://gist.github.com/5378807

As you can see, the consts of *EventID* and *Category* are stored in the filter's implementation as the original EventLogAppender uses inline consts to check them. Once their added they will be used by the mentioned *EventLogAppender* to mark the given entries with EventId and Category. The logic of determining them is up to you. You can hash the exception stacktrace, use [an exception's source property](http://msdn.microsoft.com/en-us/library/system.exception.source.aspx) or something else. At the very end remember to add this filter to your appender. Happy logging! ;-)
