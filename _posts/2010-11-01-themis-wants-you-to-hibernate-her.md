---
layout: post
title: "Themis wants you to hibernate her!"
date: 2010-11-01 20:00
author: scooletz
permalink: /2010/11/01/themis-wants-you-to-hibernate-her/
nocomments: true
categories: ["Design patterns", "NHibernate", "Themis"]
tags: ["design patterns"]
imported: true
---

Recently, I've been working on project which in the near future will have a quite complex authorization rules. Additionally, these rules will affect the display, simply filtering data sets, which one can view. Instantly I thought about [Themis](http://themis.codeplex.com/ "http://themis.codeplex.com/") and it's 'future feature' allowing to integrate with NH. What I'd like to have is simple Themis' role definition:

```csharp
public class AnalystRoleDefinition : BaseRoleDefinition
{
    public AnalysRoleDefinition( )
    {
        // CanView - helper method introduced in the application, wrapping Themis': Add blah blah
        CanView<IProtectedDocument>((document, analyst)=> document.ProtectionLevel <= analyst.AllowedProtectionLevel);
    }
}
```

Ok... As I can see, each analyst is given permission to view protected documents ONLY when he/she has *AllowedProtectionLevel* greater than document. According to Themis' functionality it's simple to create a service configured this way and check, whether in the context of a specified document this permission is granted. But what about filtering? If the analyst is disallowed to view such a document, shouldn't it be hidden?

#### NHibernate filters to the rescue

[NHibernate filters](http://ayende.com/Blog/archive/2009/05/04/nhibernate-filters.aspx "http://ayende.com/Blog/archive/2009/05/04/nhibernate-filters.aspx") can be used to add specific conditions to querying classes and their collections. Being given the role definition mentioned earlier I could add filter with an easy condition (the analyst property would be a parameter got from the context, the real condition is based on a document *ProtectionLevel* column) for each class implementing the *IProtectedDocument* and activate them with one call based on the context. My very first proposal for API would be:

```csharp
var roles = _roleService.GetCurrentUserRoles();
using(var filter = _nhAuthorizationService.ApplySessionFilters(_session, roles)
{
    var q = _session.CreateQuery("from IProtectedDocument");
    // other query stuff
}
```

The filter wrapped in *using* is an applier of filters which uses *ISession.EnableFilter* during creation and *ISession.DisableFilter* to undo filtering. It leave the session in a untouched filtering state.

If you bothered with explicitness and you want something implicit, you can easily add this behavior to your DI and never thing about it again.

Any thoughts about it?
