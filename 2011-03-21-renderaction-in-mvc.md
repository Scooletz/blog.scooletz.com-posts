---
layout: post
title: "RenderAction in MVC"
date: 2011-03-21 20:00
author: scooletz
permalink: /2011/03/21/renderaction-in-mvc/
nocomments: true
categories: ["ASP MVC"]
tags: []
imported: true
---

Recently I was debating with my friend about using [RenderAction](http://msdn.microsoft.com/en-us/library/system.web.mvc.html.childactionextensions.aspx) methods instead of [rendering partial views](http://msdn.microsoft.com/en-us/library/system.web.mvc.html.renderpartialextensions.aspx). One could say, what the fuzz is all about. It's all about simplicity. Rendering some part of your page with partial, makes you create big, fat model, passing its parts all around (for instance Model.SthForFirstPartial, Model.SthForSecondPartial). It can bring you to one model per one view or even action and this is baaaad.
You have an alternative: you can simply render actions results in you parent view using mentioned methods. Advantages:

* only atomic values passed (for instance, post id to render its comments)
* atomic actions (like: Comments, a list of comments) with results rendering not full-blow html, but parts of it (JSON is also applicable in here)

It seems, that this would be all, but it is not. Using NHibernate, my favourite ORM, and having it instantiated once per request (for Windsor fans: LifestyleType.PerWebRequest) I can easily **fetch** needed data in the parent action, loading it in a session first level cache. The child action then, will simply get data already fetched, having no problems with getting to many hits on your db.

I do like this paradigm.

### For disassemblers

(the people, which overuse Reflector, or its free replacements, to read code), there's a few lines of code, which passes the original HttpContext with all its Items (yeah, that's the base for having LifestyleType.PerWebRequest working), an excerpt from [ChildActionExtensions
](http://msdn.microsoft.com/en-us/library/system.web.mvc.html.childactionextensions.aspx) class:
```csharp
var httpContext = htmlHelper.ViewContext.HttpContext;
var context = new RequestContext(httpContext, data2); // whoa! I'll get my request items back!
var httpHandler = new ChildActionMvcHandler(context);
httpContext.get_Server().Execute(HttpHandlerUtil.WrapForServerExecute(httpHandler), textWriter, true);
```
