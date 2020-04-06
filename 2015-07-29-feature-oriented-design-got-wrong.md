---
layout: post
title: "Feature oriented design got wrong"
date: 2015-07-29 11:00
author: scooletz
permalink: /2015/07/29/feature-oriented-design-got-wrong/
categories: ["Architecture", "Design"]
tags: ["architecture", "design", "design patterns", "feature toggle"]
imported: true
---

The fourth link in my google search for 'feature toggle' is a link to this [Building Real Software post](http://swreflections.blogspot.com/2014/08/feature-toggles-are-one-of-worst-kinds.html). It's about not about feature toggles described by [Martin Folwer](http://martinfowler.com/bliki/FeatureToggle.html). It's about feature toggles got wrong.

If you consider toggling features with flags and apply it literally, what you get is a lot of branching. That's all. Some tests should be written twice to handle a positive and a negative scenario for the branch. The reason for this is a design not prepared to handle toggling properly. In the majority of cases, it's a design which is not feature-based on its own.

The featured based design is created on the basis of closed components, which handle the given domain aspect. Some of them may be big like 'basket', some may be much smaller, like 'notifications' reacting to various changes and displaying needed information. The important thing is to design the features as closed components. Once you have it done this way, it's easier to think about the page without notifications or ads. Again, disabling the feature is not a mere flag thrown in different pieces of code. It's disabling or replacing the whole feature.

One of my favorite architecture styles, [event driven architecture](http://martinfowler.com/eaaDev/EventNarrative.html) helps in a great manner to build this kind of toggles. It's quite easy to simply... not handle the event at all. If you consider the notifications, if they are disabled, they simply do not react to various events like 'order-processed', etc. The separate story is to not create cycles of dependencies, but still, if you consider the reactive nature of connections between features, that's a great enabler for introducing toggling with all of advantages one can derive from it with A/B tests, canary releases in mind.

I'm not a fan boy of feature toggling, I consider it as an important tool in architects arsenal though.
