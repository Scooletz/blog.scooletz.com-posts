---
layout: post
title: "NHibernating with interfaces as entities, pt. 2"
date: 2010-12-01 20:00
author: scooletz
permalink: /2010/12/01/nhibernating-with-interfaces-as-entities-pt-2/
nocomments: true
categories: ["C#", "NHibernate"]
tags: ["dependency injection"]
imported: true
---

The solution to the question, what can go wrong with NHibernate, when you use interfaces as your entities mapped with NH is... **naming**.

When you try to save a transient object, to make it persistent with NH, you call **ISession.Save(object)** method. If you drilled down its implementation you'd see that it fires NH's event **SaveOrUpdateEvent** which finally, after calling the default event handler for this event, would try to get the persister of the entity being saved. What for? Because you've got to know how you are trying to make persistent and later, saved in a db.

Let's take a look at the implementation provided with a  **SessionImpl.GetEntityPersister** method. At the very first lines of code, the method calls another session's method called  **GuessEntityName**. Guessing method firstly delegates the question to the interceptor, and if one was kind enough to help it to deal with it, it returns the result of the interceptor's call. Otherwise... **GetType** of the entity method is called! If the result of resolving entity's interface from container was wrapped with a proxy, some proxy type will be returned (in DynamicProxy it will be nicely placed in a dynamic assebly with a cute namespace) and because it is not what was mapped, an error will occur!

The only solution is to traverse all interfaces implemented by the object and find the one you've written in your mappings in the interceptor method. You can do it simply with the code presented below, initializing the finder with all interfaces of all your entities. Remember to do not cross trees in a forest of your interface-entity derivations:P

```csharp
/// <summary>
/// The interface finder allows you to find the deepest interface from a specified set of interfaces.
/// </summary>
/// <remarks>
/// The interfaces inheritance graph should be a forest.
/// </remarks>
public class InterfaceFinder : IDisposable
{
	private readonly List<Type> _interfaces;
	private readonly Dictionary<Type, int> _interfaceToLevel;
	private readonly Dictionary<Type, Type> _cacheObjectTypeToInterface;
	private readonly ReaderWriterLockSlim _rwl;

	public InterfaceFinder(IEnumerable<Type> interfaces)
	{
		if (!interfaces.Any())
		{
			throw new ArgumentException("No interfaces passed", "interfaces");
		}

		_interfaces = interfaces.ToList();
		_interfaceToLevel = new Dictionary<Type, int>();
		_cacheObjectTypeToInterface = new Dictionary<Type, Type>();

		// prepare dictionary counting ancestors
		foreach (var i in interfaces)
		{
			if (!i.IsInterface)
			{
				throw new ArgumentException(string.Format("The type {0} is not an interface", i.FullName));
			}

			var ancestorCount = i.GetInterfaces().Count(t => interfaces.Contains(t));
			_interfaceToLevel[i] = ancestorCount;
		}

		// create only when fully initialized
		_rwl = new ReaderWriterLockSlim();
	}

	/// <summary>
	/// Gets the depest interface from inheritance trees retrieved from
	/// the constructor parameter.
	/// </summary>
	/// <param name="o">The object to be searched for interfaces.</param>
	/// <returns>The deepest interface of a hierachy found in the object interfaces.</returns>
	public Type GetDeepestInterface(object o)
	{
		o.ThrowIfNull("o");

		var type = o.GetType();

		// enter read lock, try found, multiple threads can enter
		_rwl.EnterReadLock();
		try
		{
			Type result;
			if (_cacheObjectTypeToInterface.TryGetValue(type, out result))
			{
				return result;
			}
		}
		finally
		{
			_rwl.ExitReadLock();
		}

		// write lock, cause no entry found, one thread can enter
		_rwl.EnterWriteLock();
		try
		{
			Type result;
			if (_cacheObjectTypeToInterface.TryGetValue(type, out result))
			{
				return result;
			}

			result = GetDeepestInterface(type);
			_cacheObjectTypeToInterface[type] = result;
			return result;
		}
		finally
		{
			_rwl.ExitWriteLock();
		}
	}

	private Type GetDeepestInterface(Type objType)
	{
		var interfaces = objType.GetInterfaces().Intersect(_interfaces);

		var levels = interfaces.Select(t => _interfaceToLevel[t]).ToArray();
		var countDistinct = levels.Distinct().Count();

		if (countDistinct != levels.Length)
		{
			throw new InvalidOperationException(
				"The type of passed object implements to many interfaces, " +
			"disallowing to find one path in derivation tree. The found implemented interfaces are: " +
			string.Join(", ", interfaces.Select(i => i.FullName).ToArray()));
		}

		// find the interface with the biggest depth
		return interfaces.OrderByDescending(t => _interfaceToLevel[t]).First();
	}

	public void Dispose()
	{
		_rwl.Dispose();
	}
}
```

Tests not included :P
