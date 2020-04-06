---
layout: post
title: "An action provider, other than controller"
date: 2011-05-25 11:00
author: scooletz
permalink: /2011/05/25/an-action-provider-other-than-controller/
nocomments: true
categories: ["ASP MVC", "Design"]
tags: ["C#"]
imported: true
---

Yep, that's another post in a series which will finally bring so-called detached actions to live. Today I'll introduce the concept of action providers, something, which in my opinion is missing in the ASP MVC.

The standard pipeline of handling a request in ASP MVC starts with creating a controller with implementation of *IControllerFactory*. The default one, uses simply a controller constructor, passing none to it. Then controller is returned and its *Controller.ActionInvoker* is queried for a method/action matching the request. If none is found, an exception is thrown. After reading this paragraph it must be clear, that adding an action to a controller can be done also on the action invoker level.

The *IActionInvoker* interface provides not much space to move, since it has only one, bool method. If we take a look into the default implementation, *ControllerActionInvoker*, we can find an interesting method called: *FindAction*. The method returns something called *ActionDescriptor* and in the default implementation scans only the controller type in the search of the actions. But we can *describe* some static methods and return them during the search. To address it, I introduced an *IActionProvider* interface, which can be (almost) easily implemented.

To get the whole idea, take a look into GIST: [https://gist.github.com/989733](https://gist.github.com/989733)

As always, all needed types should be registered in the Windsor Container and resolved with it (in the current example, the only need which is to be resolved is IControllerFactory, or nothing if you register service location with the new feature from MVC 3).

Hope you like it. In the very next post, the final implementation of one of IActionProviders.
