---
layout: post
title: "Simple Cassandra backups"
date: 2014-02-10 10:00
author: scooletz
permalink: /2014/02/10/simple-cassandra-backups/
nocomments: true
categories: []
tags: ["backup", "Cassandra", "NoSql", "simplicity"]
imported: true
---

Cassandra is one of the most interesting NoSQL databases which resolves plenty of complex problems with extremely simple solutions. They are not the easiest options, but can be deduced from this db foundations.
Cassandra uses Sorted String Tables as its store for rows values. When queried, it simply finds the value offset with the index file and searched the data file for this offset. New files are flushed once in a while to disc and a new memory representation of SST is started again. The files, once stored on disc are no longer modified (the compactation is another scenario). How would you backup them? Here comes the simplicity and elegance of this solution. Cassandra stores hard links to each SST flushed from memory in a special directory. Hard links preserves removing of a file system inodes, allowing to backup your data to another media. Once once backup them, they can be removed and it'd be the file system responsibility to count whether it was the last hard link and all the inodes can be set free. Having your data written once into not modified files gives you this power and provides great simplicity. That's one of the reasons I like Cassandra's design so much.

The docs for Cassandra backups are available [here](http://www.datastax.com/documentation/cassandra/2.0/webhelp/cassandra/operations/ops_backup_restore_c.html).
