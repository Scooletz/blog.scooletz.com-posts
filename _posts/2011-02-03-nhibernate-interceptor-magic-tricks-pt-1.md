---
layout: post
title: "NHibernate interceptor magic tricks, pt. 1"
date: 2011-02-03 20:00
author: scooletz
permalink: /2011/02/03/nhibernate-interceptor-magic-tricks-pt-1/
nocomments: true
categories: ["NHibernate"]
tags: []
imported: true
---

NHibernate 3.0 is [here](http://nhforge.org/)! I've pushed it to a few solutions I'm working on as soon as it became stable. There are some nice changes (for instance polymorphic loads and gets and *finally* LINQ with the Remotion backup) but there are some core elements, which still are not well understood in the society of NH users (at least, I meet guys not understanding it). The very first candidate for that is an all mighty IInterceptor, which can change many NHibernate's behaviors. Below you can find its interface copied:

```csharp
public interface IInterceptor
{
    void AfterTransactionBegin(ITransaction tx);
    void AfterTransactionCompletion(ITransaction tx);
    void BeforeTransactionCompletion(ITransaction tx);
    int[] FindDirty(object entity, object id, object[] currentState, object[] previousState, string[] propertyNames, IType[] types);
    object GetEntity(string entityName, object id);
    string GetEntityName(object entity);
    object Instantiate(string entityName, EntityMode entityMode, object id);
    bool? IsTransient(object entity);
    void OnCollectionRecreate(object collection, object key);
    void OnCollectionRemove(object collection, object key);
    void OnCollectionUpdate(object collection, object key);
    void OnDelete(object entity, object id, object[] state, string[] propertyNames, IType[] types);
    bool OnFlushDirty(object entity, object id, object[] currentState, object[] previousState, string[] propertyNames, IType[] types);
    bool OnLoad(object entity, object id, object[] state, string[] propertyNames, IType[] types);
    SqlString OnPrepareStatement(SqlString sql);
    bool OnSave(object entity, object id, object[] state, string[] propertyNames, IType[] types);
    void PostFlush(ICollection entities);
    void PreFlush(ICollection entities);
    void SetSession(ISession session);
}
```

In a few following posts I'll try delve into the IInterceptor guts and write a nice description of its methods.

### To be singleton or not to be

The very first thing you should now about an interceptor is its lifetime. There are two possibilities: it can be either singleton or an instance attached to one session (that in majority of cases implies  *per request lifetime*). If you want to have an interceptor as a singleton, then you should set the *Configuation.Interceptor* property with an instance of your interceptor you created. Otherwise, if you want it to be bound to one session, you should create your sessions with *ISessionFactory.OpenSession(IInterceptor sessionLocalInterceptor)*. In majority of cases, creating interceptors on per request/per session basis was much more helpful to me than having a rigid singleton object.

### Getting your session

Once you decide to create an interceptor on per request basis, each time you create a new session, passing an interceptor instance, the *IInterceptor.SetSession(ISession session)* method is called. It's the right time to save the instance of your session for future use. So you have your interceptor initialized and ready to work for you.

### Am I transactional enough?

As you could see, there are three methods dealing with transactions. They are:

* *AfterTransactionBegin(ITransaction tx)*
* *BeforeTransactionCompletion(ITransaction tx)*
* *AfterTransactionCompletion(ITransaction tx)*

Once you create a transaction, you can be sure that *AfterTransactionBegin(ITransaction tx)* is called. If you throw an exception in this method, it will not be swallowed and propagate, stopping your code from execution. The next method, *BeforeTransactionCompletion(ITransaction tx)* is called just before committing a transaction. It's the last moment when you can alter your db in a safe, transactional manner. This method was very helpful in a few projects, when after analyzing changes during a session flush, some additional db operations were made. There is one catch. If you throw an exception, it will be logged and... swallowed. No regular throwing up in here mate! (If there is a need of using code which can cause an exception, there is an alternative though: *ActionQueue.RegisterProcess(BeforeTransactionCompletionProcessDelegate process)*.) The very last method, *AfterTransactionCompletion(ITransaction tx)*, is called when a commit was already done. One more time, throwing an exception will be logged and hidden.

### Summing up

Now you should be able to create a simple interceptor, which can store a reference to a session which uses it. Additionally, you know all the IInterceptor's methods dealing with transactions. In the very next posts more about it!
