---
layout: post
title: "NHibernate interceptor magic tricks, pt. 3"
date: 2011-02-14 23:00
author: scooletz
permalink: /2011/02/14/nhibernate-interceptor-magic-tricks-pt-3/
categories: ["NHibernate"]
tags: []
imported: true
---

The very [last meeting](http://blog.scooletz.com/2011/02/04/nhibernate-interceptor-magic-tricks-pt-2/) with *IInterceptor* was ended by description of *IInterceptor.GetEntityName(object entity);* method. It's time to step into the kingdom of almighty DI and IoC and learn how to combine it with NHibernate.

### Instantiate me, now!

Imagine that you have a dependency passed as your entity constructor. Or you have some methods or properties which should be called immediately after one entity construction. (It's out of the scope of this blog whether injection dependencies in your entities is good or bad. You can ask Google and check whether one or another solution fits for you.) One of the proposals of having an entity class with a non default constructor is providing a custom *IByteCodeProvider*. You can find this solution in [here](http://nhforge.org/blogs/nhibernate/archive/2008/12/12/entities-behavior-injection.aspx) but I would recommend another one, if only entity creation bothers you (the mentioned *IByteCodeProvider* initializes much more NHibernate elements than you can imagine). The second approach is to use *IInterceptor.Instantiate(string entityName, EntityMode entityMode, object id)* method. As it was shown by [Ayende](http://ayende.com/blog/default.aspx) in his [MSDN article](http://msdn.microsoft.com/en-us/magazine/ee819139.aspx), the code should look like (I use in the example Unity container interface):

```csharp
public class DependencyInjectionInterceptor : EmptyInterceptor
{
    private readonly IUnityContainer _container;
    private ISession _session;

    public DependencyInjectionInterceptor (IUnityContainer container)
    {
        _container = container;
    }

    public void SetSession(ISession session)
    {
        _session = session;
    }

    public override object Instantiate(string clazz, EntityMode entityMode, object id)
    {
        if(entityMode == EntityMode.Poco)
        {
            var type = Type.GetType(clazz);
            if (type != null)
            {
               var instance = _container.Resolve(type);
               var md = _session.SessionFactory.GetClassMetadata(clazz);
               md.SetIdentifier(instance, id, entityMode);
               return instance;
            }
        }
        return base.Instantiate(clazz, entityMode, id);
  }
}
```
You can consider passing the *ISessionFactory* in the constructor as dependency, but as I tend to use *IInterceptor* one per session, I simply use the *SetSession* method and further, the *SessionFactory* property.

That would be all about a simple case of dependency injection with NHibernate (creating only your entities). It seems that there will be a few posts more about IInterceptor stuff ;-)
