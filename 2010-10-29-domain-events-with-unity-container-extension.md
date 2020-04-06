---
layout: post
title: "Domain events with Unity Container extension"
date: 2010-10-29 20:00
author: scooletz
permalink: /2010/10/29/domain-events-with-unity-container-extension/
categories: ["Architecture", "Design patterns"]
tags: ["architecture", "design patterns"]
imported: true
---

I do like domain events perfectly described by [Udi](http://www.udidahan.com/2009/06/14/domain-events-salvation/ "http://www.udidahan.com/2009/06/14/domain-events-salvation/"). I'd like to share my implementation of this pattern, using [Unity](http://unity.codeplex.com/ "http://unity.codeplex.com/") (my preferable container), which seems to be simple, short and still powerful. I assume, that you read the Udi's article carefully. First and foremost: the domain interfaces

```csharp
    /// <summary>
    /// The markup interface for any system event.
    /// </summary>
    public interface IEvent
    {
    }

    /// <summary>
    /// The interface of a handler for events of type <typeparamref name="T"/>.
    /// </summary>
    /// <typeparam name="T">The type of the event to handle.</typeparam>
    public interface IEventHandler
        where T : IEvent
    {
        /// <summary>
        /// Handles the specified @event.
        /// </summary>
        /// <param name="event">The @event.</param>
        void Handle(T @event);
    }

    /// <summary>
    /// The interface of event manager, allowing raising events based on the <see cref="IEvent"/>.
    /// </summary>
    public interface IEventManager
    {
        /// <summary>
        /// Raises the specified event, not requiring,
        /// that it is handled by any handler.
        /// </summary>
        /// <typeparam name="T">The type of the event.</typeparam>
        /// <param name="event">The @event.</param>
        /// <remarks>
        /// The method iterates through all of the registered handlers for the event of type <typeparamref name="T"/>.
        /// </remarks>
        void Raise(T @event)
            where T : IEvent;
    }
```

I assume usage of dependency injection and having the *IEventManager* implementation injected. Speaking about the *IEventManager* implementation it's fairly simple, located in a project knowing the Unity assembly (*Infrastructure* in my case). It's worth to mention, that I consider handlers to be transient objects, constructed and discarded immediately after event handling.

```csharp
    /// <summary>
    /// The implementation of event manager using the container to resolve handlers.
    /// </summary>
    public class EventManager : IEventManager
    {
        private readonly IUnityContainer _container;

        [DebuggerStepThrough]
        public EventManager(IUnityContainer container)
        {
            _container = container;
        }

        [DebuggerStepThrough]
        public void Raise(T @event)
            where T : IEvent
        {
            var handlers = _container.&lt;ResolveAll&gt;();

            foreach (var handler in handlers)
            {
                try
                {
                    handler.Handle(@event);
                }
                finally
                {
                    _container.Teardown(handler);
                }
            }
        }
    }
```

The last but not least: *how to register all the handlers in the container*? I wrote a small unity extension providing *assembly scan* functionality which can be found below. The are two mouthful method names in the code:
* *GetFirstClosedGenericInterfaceBasedOnOpenGenericInterface*, which tries to do what is written it does :P
* *RegisterInstanceWithSingletonLifetimeManager*,an extension methods using a *I-have-nothing-to-do-with-monitors* LifetimeManager for setting up the singletons

The code itself:

```csharp
    /// <summary>
    /// The <see cref="UnityContainer"/> extension registering all the event handlers
    /// and unity-based implementation of <see cref="IEventManager"/>.
    /// </summary>
    /// <remarks>
    /// The event handlers are regsitered as classes with transient lifetime.
    /// Their instances are tear down immediately after usage.
    /// </remarks>
    public class EventUnityContainerExtension : UnityContainerExtension
    {
        protected override void Initialize()
        {
            var eventManager = new EventManager(Container);
            Container.RegisterInstanceWithSingletonLifetimeManager(eventManager);
        }
        /// <summary>
        /// Registers all the event handlers from assembly of the passed type <typeparamref name="T"/>.
        /// </summary>
        /// <typeparam name="T">The type which assembly should be scanned.</typeparam>
        /// <returns>This instance.</returns>
        public EventUnityContainerExtension RegisterHandlersFromAssemblyOf<T>()
        {
            return RegisterHandlersFromAssembly(typeof(T).Assembly);
        }

        /// <summary>
        /// Registers all the event handlers from all <paramref name="eventHandlersAssemblies"/>.
        /// </summary>
        /// <param name="eventHandlersAssemblies">List of assemblies to scan.</param>
        /// <returns>This instance.</returns>
        public EventUnityContainerExtension RegisterHandlersFromAssemblies(params Assembly[] eventHandlersAssemblies)
        {
            foreach (var eventHandlersAssembly in eventHandlersAssemblies)
            {
                RegisterHandlersFromAssembly(eventHandlersAssembly);
            }

            return this;
        }

        private EventUnityContainerExtension RegisterHandlersFromAssembly(Assembly assembly)
        {
            foreach (var type in assembly.GetTypes())
            {
                if (type.IsInterface || type.IsAbstract)
                {
                    continue;
                }

                var handlerInterface = type.GetFirstClosedGenericInterfaceBasedOnOpenGenericInterface(typeof(IEventHandler));
                if (handlerInterface == null)
                {
                    continue;
                }

                // the type is registered with a name, because only named registrations can be resolved by unity.ResolveAll method
                Container.RegisterType(handlerInterface, type, type.FullName, new TransientLifetimeManager());
            }

            return this;
        }
    }
}
```

Simple and powerful, isn't it? :)
