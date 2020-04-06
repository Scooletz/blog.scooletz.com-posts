---
layout: post
title: "Solrnet NHibernate integration"
date: 2010-09-14 23:36
author: scooletz
permalink: /2010/09/14/solrnet-nhibernate-integration/
categories: ["Databases", "NHibernate"]
tags: ["databases", "solr"]
imported: true
---

Every developer which creates a scalable applications with high read/write ratio finally has to move to some query oriented storage, which allows almost instant, advanced querying. [Solr](http://lucene.apache.org/solr/ "Solr") is one of the frequently used solutions, providing a nice performance with advanced indexing/querying. Talking to Solr needs an API and for .NET and you can use [SolrNet](http://code.google.com/p/solrnet/ "SolrNet"). As it is described on the project page, it provides a nice and simple way to integrate with NHibernate, simply by calling following code:

```csharp
NHibernate.Cfg.Configuration cfg = SetupNHibernate();
var cfgHelper = new NHibernate.SolrNet.CfgHelper();
cfgHelper.Configure(cfg, true); // true -> autocommit Solr after every operation (not really recommended)
```

The integration with NH allows you to use an extended session to query the solr server. Additionally, all your updates, inserts, deletes will be automatically called on the solr. This feature is provided by NH events listeners registered by the described configuration helper. Let's take a look and audit a bit of code:

```csharp
public Configuration Configure(Configuration config, bool autoCommit)
{
 foreach (Type type in this.mapper.GetRegisteredTypes())
 {
 Type type2 = typeof(SolrNetListener<>).MakeGenericType(new Type[] { type });
 Type serviceType = typeof(ISolrOperations<>).MakeGenericType(new Type[] { type });
 object service = this.provider.GetService(serviceType);
 ICommitSetting listener = (ICommitSetting) Activator.CreateInstance(type2, new object[] { service });
 listener.Commit = autoCommit;
 this.SetListener(config, listener);
 }
 return config;
}
```

For each of mapped types a new listener is generated. Is it OK? Why do not use one listener (with no generics at all), handling all the mapped types?
Additionally, doesn't SetListener method clear all the previously registered listeners, so... what about the types previously handled?

The more interesting question can be raised when looking through the [SolrNetListener<T>](http://github.com/mausch/SolrNet/blob/master/NHibernate.SolrNet/Impl/SolrNetListener.cs "SolrNetListener<t>")</t> code. All the listeners in NH are singletons. They are initialized during the start up phase, and used till the application end. Hence, listeners should be stateless, or use some kind of current context resolvers passed in their constructor. The SolrNetListener uses WeakHashtables fields to store the entities which should be flushed to the solr. Static field (because the listener is a singleton)? What about the race conditions, locking etc.? Let's take a look:

The example of Delete method, called, when solr delete should be deferred (session uses a transaction) shows that listener GLOBALLY locks the execution of all threads:

```csharp
private void Delete(ITransaction s, T entity)
{
 lock (this.entitiesToDelete.SyncRoot)
 {
 if (!this.entitiesToDelete.Contains(s))
 {
 this.entitiesToDelete[s] = new List<T>();
 }
 ((IList<T>) this.entitiesToAdd[s]).Add(entity);
 }
}
```

furthermore, data can be committed to the solr server, but not committed to the sql database! It can happen when flushing occurs. The override of OnFlush method saves the data to the solr before flushing the db changes.
What if optimistic locking rolls back the whole transaction after the data were stored in the solr? I'd rather have no data in solr and use some background worker to upsert data in solr then have copies of nonexisting entities.

The right behavior for the solr, with no global locks, flushing changes to the solr after a sql db transaction commit can be done with using an Interceptor and rewritten the event handler. I'll describe it in the next post.

Regards
