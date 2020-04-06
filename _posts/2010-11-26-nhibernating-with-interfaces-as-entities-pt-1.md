---
layout: post
title: "NHibernating with interfaces as entities, pt. 1"
date: 2010-11-26 20:00
author: scooletz
permalink: /2010/11/26/nhibernating-with-interfaces-as-entities-pt-1/
nocomments: true
categories: ["C#", "NHibernate"]
tags: ["dependency injection"]
imported: true
---

No catchy title this time:P
Have you ever considered using interfaces as entities in your application with NH as ORM? If not, and you never considered this option I can share a few thoughts about it with you.

Proxifing an interface is easier than proxifing a class, especially when using a tool like DynamicProxy which as far as I know still does not has this ability (yep, I've read about it on Kozmic's blog). Why on earth proxify each entity you may ask? During the last project I was involved in, my colleague wanted to ensure, that setting properties and calling methods of one entity can be done only in explicitly set up scopes (explicit security). It's quite easy when you can intercept all the setters and other, non-property method calls. All you've got to do is during instantiation of an entity wrap it with a certain proxy. What about instantiating entities' implementations? There should be some, because it very unpleasant to have your entities only data oriented with no methods at all. This can be easily done with your implementation of NH's IInterceptor. There is an interceptor method *Instantiate(string entityName, EntityMode entityMode, object id);* which, with help of your favorite container, can be turned into a entity factory. The code would look like this:

```csharp
public override object Instantiate(string clazz, EntityMode entityMode, object id)
{
	if (entityMode == EntityMode.Poco)
	{
		var metadata = _sessionFactory.GetAllClassMetadata()[clazz];
		var type = metadata.GetMappedClass(entityMode);

		if (type != null)
		{
			var instance = _unityContainer.Resolve(type);
			var classMetadata = _sessionFactory.GetClassMetadata(clazz);

			classMetadata.SetIdentifier(instance, id, entityMode);

			return instance;
		}
	}

	return null;
}
```

The session can easily create the entity to hydrate it, we can be given our ISomeEntity with Load, Get, List.  What about creating your entity for the first time? It can be also achieved providing an interface of *IEntityFactory* implemented with usage of the very same container used in the interceptor.

So far so good. But is there a catch? Yes there is and it'll be revealed soon :)
