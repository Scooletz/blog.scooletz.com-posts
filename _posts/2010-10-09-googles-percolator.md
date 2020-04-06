---
layout: post
title: "Google's Percolator"
date: 2010-10-09 20:00
author: scooletz
permalink: /2010/10/09/googles-percolator/
nocomments: true
categories: ["Architecture", "Databases", "Google"]
tags: []
imported: true
---

Next great article from Google: [Percolator](http://research.google.com/pubs/pub36726.html# "Percolator"). The whitepaper describes a new indexing engine, which no longer uses the massive MapReduce algorithm to calculate web pages indexes. As always, Google used a few already existing tools, like BigTable and build an index updater, which opposite to MapReduce updates small chunks of the index repository drastically reducing :-) the time between a page being crawled and being indexed. Worth to read, even for the transaction scheme implemented on the non-transactional BigTable.
