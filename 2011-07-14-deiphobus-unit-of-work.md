---
layout: post
title: "Deiphobus, unit of work"
date: 2011-07-14 11:00
author: scooletz
permalink: /2011/07/14/deiphobus-unit-of-work/
nocomments: true
categories: ["Deiphobus"]
tags: ["Cassandra", "NoSql"]
imported: true
---

You can find a very nice description of *Unit of Work* in [here](http://martinfowler.com/eaaCatalog/unitOfWork.html "Unit of work"). It describes a mythological artifact, which can understand what you're doing with object retrieved from a database and allows you easily persist the whole state back to the database. Typically, clients for NoSQL databases do not provide this high abstraction of the persistence. [Deiphobus](https://github.com/Scooletz/Deiphobus) makes a difference, providing a well known from other object mappers, the mighty unit of work, called *Session*.  Below you can find how simple example of the session usage is:

```csharp
// sessionFactory created earlier from a configuration
using (var s = sessionFactory.Open())
{
	// entity creation makes it tracked, the state will be saved at s.Flush();
	var user = s.Create<IUser>(t =>
	{
		t.Email = "Cassandra@cassandra.org";
		t.AllowMarketingEmails = true;
	});

	var secret = s.Create<IPersonalInfo>(t =>
	{
		t.Type = PersonalInfoType.SecretFact;
		t.Body = "I dislike SQL paradigm";
	});

	var quote = s.Create<IPersonalInfo>(t =>
	{
		t.Type = PersonalInfoType.Quote;
		t.Body = "To be SQL or not to be";
	});

	user.Infos.Add(secret);
	user.Infos.Add(quote );

	// Flush to persist all changes at once! Dirty checks, etc. are handled without your coding!
	s.Flush();
}
```

It is simple and not dealing with checking what should be saved. Just create a session and call *Flush* when you want your changes be persisted. Of course it isn't transactional as the database [Cassandra](http://cassandra.apache.org/ "Cassandra home page") isn't, but still, it puts a nice layer over the standard Thrift protocol.
