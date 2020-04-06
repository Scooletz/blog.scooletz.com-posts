---
layout: post
title: "Overvalidated design"
date: 2012-12-10 10:00
author: scooletz
permalink: /2012/12/10/overvalidated-design/
nocomments: true
categories: ["ASP MVC", "C#", "Design"]
tags: []
imported: true
---

Imagine, that you're requested to allow adding additional validation in an ASP MVC application to properties of some models. The set of properties' types is given and contains following types: string, int, DateTime, a reference to a glossary entity. The business (why?) scenario is irrelevant and will be not exposed here.
To allow MVC work nicely there are multiple ways to resolve this case. The following ways may be used:

* a custom [ModelValidatorProvider](http://msdn.microsoft.com/en-us/library/system.web.mvc.modelvalidatorprovider.aspx) implementation can be provided, which takes the validation information saved in some storage and applies to
* an extension to [TypeDescriptor](http://msdn.microsoft.com/en-us/library/system.componentmodel.typedescriptor.aspx) facilities, to add attributes to the specified properties

But how to store this information, being given a meta description (types and their properties already mapped to entities in a database). The first take was to provide an abstract class *Validator* and try to subclass it with a fully descriptive design, allowing saving any type of parameter, with any kind of operator. You can imagine how many enums, objects (not so strongly typed API) it created.

The *"what for?"* question arose. Being given a set of types and possibilities of their validation why not to provide validators tightly coupled to those types? If the set of validated types is frozen, the switches can be easily replaced with visitors (very stable hierarchy), which can easily transform the given data into sth which may be used by MVC.

![Validators data new design](http://yuml.me/43c7174c)

Having your information about validators correlated with types, which should not be changed in a while, allows you for easier editing and storing (no more operators, objectly typed properties). The transformed values can be easily applied via *ModelValidator* or added as attributes to the *TypeDescriptor* of the given model. This approach creates a simple pipeline with a possibility of getting the procession result in any moment and inject it in the framework (ASP MVC) in a preferable way.
