---
layout: post
title: "Event sourcing for DBAs"
date: 2016-11-09 09:55
author: scooletz
permalink: /2016/11/09/event-sourcing-for-dbas/
nocomments: true
image: /img/2016/11/kings-row.jpg
categories: ["Databases", "Event sourcing"]
tags: ["database", "event sourcing"]
imported: true
---

**TL;DR**

Are you in a team trying to convince a database administrator  to use event sourcing? Are you a DBA that is being convinced? Or maybe it's you and a person that does not want to change the relational point of view. In any case, read forward.

### Drop. Every. Single. Table

That's the argument you should start with. You can drop every single table and your database will be just right. Why is that? There's a recovery option. If you don't run your database with some silly settings, you can just recover all the tables with all the data. Alright, but where do I recover from? What's the source of truth if I don't have tables.

### Log

The tables are just a cache. All the data are stored in the transaction file log. Yes, the file that is always to big. If all the data are in there, can it be queried somehow?

*SELECT * FROM fn_dblog(null, null)*

That's the query using an undocumented SQL Server function. It simply reads a database log file and displays it as a nicely formatted table like this:

![dblog](/img/2016/11/dblog.png)

Take a look at the operation column now. What can you see?

* LOP_INSERT_ROWS
* LOP_MODIFY_ROW
* LOP_MODIFY_COLUMNS
* LOP_COMMIT_XACT
* LOP_BEGIN_XACT
* LOP_COMMIT_XACT

What are these? That's right. That's every single change that has been applied to this database so far. If you spent enough of time with these data, you'd be able to decode  payloads or changes applied to the database in every single event. Having this said, it looks like the database itself is built with an append only store that saves all the changes done to the db.

### Event Sourcing

Event sourcing does exactly the same. Instead of using database terms and names of operations, it incorporates the business language, so that you won't find LOP_MODIFY_COLUMNS with a payload that needs to be parsed, but rather an event with an explicit business related name appended to a store. Or to a log, if you want it call it this way. Of course there's a cost of making tables out of it once again, but it pays back by pushing the business understanding and meaning much deeper into the system and bringing the business, closer to the system.

At the end these tables will be needed as well to query something. They won't be treated as a source of truth, they are just a cache anyway, right? The truth is in the log.
