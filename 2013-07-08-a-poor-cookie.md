---
layout: post
title: "A poor cookie"
date: 2013-07-08 10:00
author: scooletz
permalink: /2013/07/08/a-poor-cookie/
nocomments: true
categories: ["Html"]
tags: ["cookie", "html", "http", "javascript"]
imported: true
---

The implementation of a http cookie is leaky. Better get used to it. You can read RFCs about, but better read one, more meaningful question posted on [the security stackexchange](http://security.stackexchange.com/questions/12412/what-cookie-attacks-are-possible-between-computers-in-related-dns-domains-exa). If your site is hosted as a subdomain with others apps and a malicious user can access any other app a cookie with top domain can be set. What it means, is that the cookie will be sent with every request to the top domain as well as yours (*domain-match* verb in the RFCs). This can bring a lot of trouble when an attacker sets a cookie with a name important for your app, like a session cookie. According to the specification, both values will be sent under the same name with no additional information about on which basis a given value was sent.

### Html5 to the rescue

If you design a new Single Page Application, you can be saved. Imagine that during POST sending the login data (user & password) in a result JSON a value previously stored in cookie is returned. One can save it in the [localStorage](http://diveintohtml5.info/storage.html "localStorage") easily and add later on to the headers of requests needing authentication. A simple change brings another advantage. Requests not needing authentication like GETs (as noone sends fragile data with verb that is vulnerable to [JSON Hijacking](http://haacked.com/archive/2009/06/24/json-hijacking.aspx "json hijacking")) can be sent with no header overhead to the same domain. A standard solution to stop sending cookies with GETs is shipping all your static files to another domain. That isn't needed anymore.
