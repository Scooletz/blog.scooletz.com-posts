---
layout: post
title: "NHibernate interceptor magic tricks, the example"
date: 2011-02-24 20:00
author: scooletz
permalink: /2011/02/24/nhibernate-interceptor-magic-tricks-the-example/
nocomments: true
categories: ["Enterprise library", "NHibernate", "Unity Container"]
tags: ["Unity", "Unity container"]
imported: true
---

Below, you can find the final example of working interceptor, which uses some methods described in text of the last few blog entries ([1](http://blog.scooletz.com/2011/02/03/nhibernate-interceptor-magic-tricks-pt-1/), [2](http://blog.scooletz.com/2011/02/04/nhibernate-interceptor-magic-tricks-pt-2/), [3](http://blog.scooletz.com/2011/02/14/nhibernate-interceptor-magic-tricks-pt-3/), [4](http://blog.scooletz.com/2011/02/18/nhibernate-interceptor-magic-tricks-pt-4/) and [5](http://blog.scooletz.com/2011/02/22/nhibernate-interceptor-magic-tricks-pt-5/)). Scan the example and go below to get some explanation about it!

```csharp
public class ExampleInterceptor : EmptyInterceptor
{
	private const int MaxStatements = 50;
	private static readonly ILog Logger = LogManager.GetLogger(typeof(UpdateInterceptor));

	private readonly InterfaceFinder _interfaceFinder;
	private readonly IUnityContainer _container;
	private ISession _session;

	private int _statementCount;

	public UpdateInterceptor(IUnityContainer container, InterfaceFinder interfaceFinder)
	{
		_container = container;
		_interfaceFinder = interfaceFinder;
	}

	public override void PostFlush(ICollection entities)
	{
		if (_session.Transaction == null)
		{
			throw new InvalidOperationException("Use transactions!");
		}
		base.PostFlush(entities);
	}

	public override void SetSession(ISession session)
	{
		base.SetSession(session);
		_session = session;
	}

	public override string GetEntityName(object entity)
	{
		if (entity == null)
		{
			return null;
		}

		var interfaceType = _interfaceFinder.GetDeepestInterface(entity);

		if (interfaceType == null)
		{
			return null;
		}

		return interfaceType.FullName;
	}

	public override SqlString OnPrepareStatement(SqlString sql)
	{
		if (_statementCount++ == MaxStatements)
		{
			Logger.WarnFormat("Max number of statements exceeded");
		}

		return base.OnPrepareStatement(sql);
	}

	public override object Instantiate(string clazz, EntityMode entityMode, object id)
	{
		if (entityMode == EntityMode.Poco)
		{
			var sessionFactory = _session.SessionFactory;
			var metadata = sessionFactory.GetAllClassMetadata()[clazz];
			var type = metadata.GetMappedClass(entityMode);

			if (type != null)
			{
				var instance = _container.Resolve(type);
				var classMetadata = sessionFactory.GetClassMetadata(clazz);

				classMetadata.SetIdentifier(instance, id, entityMode);

				return instance;
			}
		}

		return null;
	}
}
```

### The constructor

As you've noticed, there is a dependency injection in here! Two arguments are: [unity container](http://unity.codeplex.com/) instance; interface finder, which allows you to use interfaces with their implementation hierarchies. About the second, you can read [here](http://blog.scooletz.com/2010/11/26/nhibernating-with-interfaces-as-entities-pt-1/) and [here](http://blog.scooletz.com/2010/12/01/nhibernating-with-interfaces-as-entities-pt-2/).

### Post flush

does nothing more than ensuring that you're running it in a transaction. Yep, one for all, all for one!

### SetSession

remembers the session instance in a field.

### GetEntityName

implementation indicates that there are some interfaces mapped, for instance IA and IB : IA. It allows the most nested interface to be easily find for the object type.

### OnPrepareStatement

preserves a sane number of statements per session (hence, per request, because session per request scenario is considered).

### Instantiate

is the final method. It uses the passed container to create an instance of the passed class. Having interfaces mapped, it's must have since you cannot call *new* for interface :P

 **Unity registration**
Having this interceptor we need a nice and easy way of registering any interceptor (which type is hold in *_interceptorType* field) in the container. That's performed by the following unity extension:

```csharp
public class NhUnityContainerExtension : UnityContainerExtension
{
	protected override void Initialize()
	{
		// ...
		// save configuration to container for any later use
		Context.Container.RegisterInstanceWithSingletonLifetimeManager(cfg);

		// build factory and register interceptor
		var factory = cfg.BuildSessionFactory();
		Context.Container.RegisterInstanceWithSingletonLifetimeManager(factory);
		Context.Container.RegisterTypeWithPerRequestLifetimeManager(typeof(IInterceptor), _interceptorType);

		var key =  NamedTypeBuildKey.Make<ISession>();
		// setup nhibernate session build plan policy
		Context.
			Policies.
			Set<IBuildPlanPolicy>(
				new DelegateBuildPlanPolicy(
					ctx =>
							{
								// create interceptor already registered
								var interceptor = BuilderContext.NewBuildUp<IInterceptor>(ctx);
								var buildUpFactory =
									BuilderContext.NewBuildUp<ISessionFactory>(ctx);
								return buildUpFactory.OpenSession(interceptor);
							}),
				key);

		// setup lifetime policy
		Context.
			Policies.
			Set<ILifetimePolicy>(CreatePerRequestLifeTimeManager(), key);
	}

	private LifetimeManager CreatePerRequestLifeTimeManager()
	{
		// ...
	}
}
```

If you know the architecture of Unity, this extension is pretty safe explanatory, event mine extension methods.

That's the end of Interceptor journey. Happy Intercepting!
