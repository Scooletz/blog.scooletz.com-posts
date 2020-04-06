---
layout: post
title: "You don't mess with Unity's policies"
date: 2010-12-21 20:00
author: scooletz
permalink: /2010/12/21/you-dont-mess-with-unitys-policies/
nocomments: true
categories: ["C#", "Optimization", "Unity Container"]
tags: ["optimization", "performance", "Unity", "Unity container"]
imported: true
---

Recently I ran into a problem. Quite well designed system degraded in terms of performance after a few commits. The whole architecture is wired with an infrastructure library written with unity container as a base for dependency injection. One of its features is a simple adding proxies, for instance adding some information to an exception's data in case of throwing one. The interceptor doing it is quite simple:
```csharp
public class ExceptionDataAppendingInterceptor : IInterceptor
{
	public void Intercept(IInvocation invocation)
	{
		try
		{
			invocation.Proceed();
		}
		catch (Exception ex)
		{
			ex.AppendMethodCallData(
			    invocation.Method, invocation.Arguments);
			throw;
		}
	}
}
```
The exension method appending data is a bit more complex but does not add any value to the post.

Having in mind that exception can be thrown at various points of an application and having it run for the very first time in the test environment, I configured unity to set proxy to the majority of created objects, which are resolved as interfaces (proxifing their interfaces).
When I run a profiled, it showed a major overhead created by one strategy, being responsible for wrapping a created object with use of DynamicProxy2. What the profiler shown was plenty of calls to IEnumerable extensions method. The strategy called one policy which before fixing looked like this:

```csharp
public class InterceptorSelectorPolicy : IBuilderPolicy
{
	private readonly IDictionary<Type, List<Func<IBuilderContext, IInterceptor>>>
		_activators;
	private static readonly ProxyGenerator ProxyGenerator = new ProxyGenerator();

	public InterceptorSelectorPolicy(
		IDictionary<Type, List<Func<IBuilderContext, IInterceptor>>> activators)
	{
		_activators = activators;
	}

	/// <summary>
	/// Determines whether the specified context is applicable for proxy generation.
	/// </summary>
	public bool IsApplicable(IBuilderContext context)
	{
		return _activators.ContainsKey(BuildKey.GetType(context.OriginalBuildKey));
	}

	/// <summary>
	/// Creates the proxy using <see cref="IBuilderContext.Existing"/> as target
	/// and <see cref="IBuilderContext.OriginalBuildKey"/> as proxied interface type.
	/// </summary>
	public object CreateProxy(IBuilderContext context)
	{
		var typeToProxy = BuildKey.GetType(context.OriginalBuildKey);

		return ProxyGenerator.CreateInterfaceProxyWithTarget(
			typeToProxy,
			context.Existing,
			_activators[typeToProxy]
			.Select(a => a(context))
			.ToArray());
	}
}
```
It's worth to mentioned that it was called **every time** an object was created... After refactorization the code lost all the enumerable _create_state_machine_for_my_iterator_ stuff and was changed to:

```csharp
public class InterceptorSelectorPolicy : IBuilderPolicy
{
	private readonly Type _typeToProxy;
	private readonly Func<IBuilderContext, IInterceptor>[] _activators;
	private static readonly ProxyGenerator ProxyGenerator = new ProxyGenerator();

	public InterceptorSelectorPolicy(Type typeToProxy,
		Func<IBuilderContext, IInterceptor>[] activators)
	{
		_typeToProxy = typeToProxy;
		_activators = activators;
	}

	/// <summary>
	/// Gets a value indicating whether this proxified should be applied.
	/// </summary>
	/// <value>
	///     <c>True</c> if this instance is applicable; otherwise, <c>false</c>.
	/// </value>
	public bool IsApplicable
	{
		get { return _activators != null && _activators.Length > 0; }
	}

	/// <summary>
	/// Creates the proxy using <see cref="IBuilderContext.Existing"/> as target
	/// and <see cref="IBuilderContext.OriginalBuildKey"/> as proxied interface type.
	/// </summary>
	/// <param name="context">The context.</param>
	/// <returns>Returns created proxy.</returns>
	public object CreateProxy(IBuilderContext context)
	{
		return ProxyGenerator.CreateInterfaceProxyWithTarget(
			_typeToProxy,
			context.Existing,
			CreateInterceptors(context));
	}

	private IInterceptor[] CreateInterceptors(IBuilderContext context)
	{
		var result = new IInterceptor[_activators.Length];
		for (var i = 0; i < _activators.Length; i++)
		{
			result[i] = _activators[i](context);
		}
		return result;
	}
}
```
No dictionary look ups, no iterators, no needed overhead. The whole idea was simplified and now, for an object having no need of proxifing it is reduced to a simple, nolock _is_null_ array check called by a post build strategy. You don't mess with Unity's policies unless you have a nice stress test written.
