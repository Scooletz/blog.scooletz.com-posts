---
layout: post
title: "Nautral identifiers as subresources in RESTful services"
date: 2015-10-14 11:00
author: scooletz
permalink: /2015/10/14/natural-identifiers-as-subresources-in-restful-services/
nocomments: true
categories: ["Design"]
tags: ["databases", "identity", "natural keys", "REST", "services"]
imported: true
---

There's a project I'm working on, which provides a great study of legacy. Beside the code, there's a database which frequently uses complex natural keys, consisting of various values. As modelling using natural complex keys may be natural, when it comes to put a layer of REST services on top of that structure, a question may be raised: how to model the identifiers in the API, should this natural keys be leaked into the API?

REST provides various ways of modelling API. One of them are REST subresources, which are represented as URIs with additional identifiers at the end. The subresource is nothing more than an identified subpart of the resource itself. Having that said, and taking as an example a simple row with complex natural key consisting of two values <Country, City> how could one model accessing cities (for sake of this example I assume that, there are cities all around the world having the same name but being in different countries and all the cities in the given country have distinct names). How one could provide a URI for that? Is the following the right one?

*/api/country/Poland/city/Warsaw*

The API shows Warsaw as the Polish city. That's true. This API has that nice notion of being easy to consume, navigate. Consider following example:

*/api/city/Poland,Warsaw*

Now it's a big uglier, both the country and the city name are at the end. This is a bit different for sure and tells nothing about country accessible under /api/country/Poland. The question is which is better?

Let me abuse a bit the [DDD Aggregate](http://martinfowler.com/bliki/DDD_Aggregate.html) term and follow its definition. Are there any operations that can be performed against the city resource/subresource that does not change the state of the country? If yes, then in my opinion modelling your API with resources shows something totally different, saying: hey, this is a city, a part of this country; it's a subresource and should be treated as a part of the country ALWAYS. Consider the second take. This one, presents a city as a standalone resource. Yes, it is identified by a complex natural key consisting of two dimentions, but this is a mere implementation detail. Once a usual identifiers like int, Guid are introduced the API won't change that much, or even better, API could accept both of them, using the older combined id for consumers that don't want to change their usage (easier versioninig).

To sum up: do not leak your internal design either it's a database design or an application design. Present your user a consistent view grouping resources under wings of transactional consistency.
