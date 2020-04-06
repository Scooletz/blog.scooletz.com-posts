---
layout: post
title: "ASP MVC Model binders"
date: 2010-11-11 16:36
author: scooletz
permalink: /2010/11/11/asp-mvc-model-binders/
categories: ["ASP MVC", "Dependency injection", "Design patterns"]
tags: ["dependency injection", "design patterns"]
imported: true
---

Recently I had to create a model for ASP MVC, which would be bound by the model binder and had its properties set according to the request. It's the most common case you can imagine, but there was one 'but'. The model, because of its nature had to have non-default constructor. It was only a few parameters but it broke the [DefaultModelBinder](http://msdn.microsoft.com/en-us/library/system.web.mvc.defaultmodelbinder%28VS.90%29.aspx "http://msdn.microsoft.com/en-us/library/system.web.mvc.defaultmodelbinder%28VS.90%29.aspx"), which simply couldn't find all the needed values. My response was quite fast, specially having the controller factory implemented in the following way:

```csharp
/// <summary>
/// The controller factory building up the controllers with the unity container.
/// </summary>
public class ControllerFactory : DefaultControllerFactory
{
    private readonly IUnityContainer _container;

    public ControllerFactory(IUnityContainer container)
    {
        _container = container;
    }

    protected override IController GetControllerInstance(RequestContext requestContext,
        Type controllerType)
    {
        return (IController)_container.Resolve(controllerType, null);
    }
}
```

The very next step was to replace the default binder stored in *ModelBinders.Binders.DefaultBinder* with another, controller-based implementation:

```csharp
/// <summary>
/// The model binder building up the model objects with the unity container.
/// </summary>
public class ModelBinder : DefaultModelBinder
{
    private readonly IUnityContainer _container;

    public ModelBinder(IUnityContainer container)
    {
        _container = container;
    }

    protected override object CreateModel(ControllerContext controllerContext,
                                          ModelBindingContext bindingContext,
                                          Type modelType)
    {
        var type = modelType;
        if (modelType.IsGenericType)
        {
            var genericTypeDefinition = modelType.GetGenericTypeDefinition();
            if (genericTypeDefinition == typeof (IDictionary<,>))
            {
                type = typeof (Dictionary<,>).MakeGenericType(modelType.GetGenericArguments());
            }
            else if (((genericTypeDefinition == typeof (IEnumerable<>)) ||
                      (genericTypeDefinition == typeof (ICollection<>))) ||
                     (genericTypeDefinition == typeof (IList<>)))
            {
                type = typeof (List<>).MakeGenericType(modelType.GetGenericArguments());
            }
        }

        // in the base method: return Activator.CreateInstance(type);
        return _container.Resolve(type, null);
    }
}
```

As you can compare, the method simply delegates creating model to the container. Having this binder set allowed passing models with non-default constructors, so the problem was solved:] After a while, I considered following case: if the type is constructed with a container, the controller can also be passed an object implementing an interface registered in container. Hence, if a method needs a NHibernate ISession, this can be expressed not only as a parameter in Controller constructor, but also as a method parameter. This paradigm creates controllers with no parameters needed in constructor (or only common parameters, used in all actions) and all the others passed as action parameters. The result is a method, which can be perfectly tested in an environment where the only smallest, needed set of dependencies is passed (no more constructors with not needed in the current method parameters). What do you think about it?

An example of a controller:
```csharp
public class CarController : Controller
{
    private readonly ISession _session;
    private const int PageSize = 10;

    public HomeController(ISession session)
    {
        _session = session;
    }

    public ActionResult Rent(IUserService userService, Guid carId)
    {
        using (var tx = _session.BeginTransaction())
        {
            var car = _session.Load<Car>(carId);
            var user = userService.GetCurrentUser();

            car.RentBy(user);

            tx.Commit();
        }

        return View();
    }

    public ActionResult Index( int? pageNo)
    {
        var page = pageNo ?? 0;

        var cars = _session.CreateQuery("from Car")
            .SetFirstResult(page*PageSize)
            .SetMaxResults(PageSize)
            .Future<Car>();

        return View(cars);
    }
}
```
