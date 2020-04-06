---
layout: post
title: "Should companies orient their IT toward service/product"
date: 2014-06-09 11:00
author: scooletz
permalink: /2014/06/09/should-big-companies-orient-their-it-toward-serviceproduct/
nocomments: true
categories: ["Agile"]
tags: ["agile", "product owner", "services", "SOA"]
imported: true
---

The question from the subject of this entry has arisen in me after a discussion with a colleague of mine. The very initial question on this subject was whether a library with contracts should be named:

1. after the consumer, embedding its name in the contract, marking it as 'it's for this app'
1. after the provider's functionality, marking it as 'this service part provides this functionality'

The answer may obvious from the development perspective. The second answer provides and abstraction over given functionality hence I prefer it over 1st. What does it mean for your organization? Well, you just have defined a contract of a service. If you have a system, with up to a few abstractions like this and a team behind it, you've got a real service, a real product owned by the team. This way of thinking in a long term may result in:

1. team's ownership feeling
1. proper contracts versioning (as it is based on)
1. top-bottom understanding of service boundaries and responsibilities
1. distilling a single API for given set of functionalities

The other way may be good as:

1. other teams may influence or dictate interfaces they need
1. versioning may be much simpler (one consumer for the contract)

Which one should choose? There's one point I didn't mention before, which shows the winner. It's the entanglement factor. The first solution introduces one functionality-one API rule and makes consumers obey the rules team wants to be obeyed, for example max number of items returned per request etc. The second is very similar to sharing the most internal parts of the system, almost like... db integration. A service team have to maintain and version multiple interfaces. Let us count it, being given

* n services
* each service depends on 3 others

when the first solution is chosen (one service-one API) the total number of published contracts is **n**. In the second case, it's **3n** and the responsibility for maintenance and fixes is blury. The second version reminds me of an entangled web with no owners. Everything is nobody's nothing.

In my opinion, the bigger shift toward service/product paradigm, with team's ownership of the product, not the code ownership, the healthier IT teams and product they make.
How about your company? Is it oriented around services?
