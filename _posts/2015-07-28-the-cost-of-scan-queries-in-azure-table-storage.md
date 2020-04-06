---
layout: post
title: "The cost of scan queries in Azure Table Storage"
date: 2015-07-28 11:00
author: scooletz
permalink: /2015/07/28/the-cost-of-scan-queries-in-azure-table-storage/
categories: ["Azure", "Azure"]
tags: ["azure", "azure table storage", "cloud", "performance"]
imported: true
---

There are multiple articles describing the performance of Azure Table Storage. You probably read the entry of Troy Hunt, [Working with 154 million records on Azure Table Storage...](http://www.troyhunt.com/2013/12/working-with-154-million-records-on.html). You may have invested your time in reading [How to get most out of Windows Azure Tables](http://blogs.msdn.com/b/windowsazurestorage/archive/2010/11/06/how-to-get-most-out-of-windows-azure-tables.aspx) as well. My question is have you really considered the limitations of the queries, specifically scan queries and how they can consume the major part of [Azure Performance Targets](https://azure.microsoft.com/en-us/documentation/articles/storage-scalability-targets/).

The PartitionKey and RowKey create the primary and the only index in ATS (Azure Table Storage). Depending on the query the following kinds can be distinguished:

1. Point Queries, which are queries to retrieve a single entity by specifying a single PartitionKey and RowKey using equality as predicate
1. Row Range Queries, which  are queries to get a set of entities defined with the same PartitionKey and a range of RowKeys
1. Partition Range Queries, which are run with a range of ParitionKeys
1. Full table scans, which have no predicate for ParitionKey

What are the costs and limitations of the following queries? Unfortunately, every row that is accessed by the query to perform scan over will be counted as the table operation, Tthere ain't no such thing as a free lunch. This means, that if you scan your entire table (4th scenario), you'll be able to process no more than 20,000 entities per second. This limits the usage of large data sets' scans. If you have to model queries across different keys, then you may consider storing the same value twice: once under the natural Parition/RowKey pair and the second time to match the other index, to create an [inverted index](https://en.wikipedia.org/wiki/Inverted_index). If any case, you'll have to scan through the entire data set, then using ATS is not the way to go, and you should consider some other ways of modelling your data, like asynchronous copy data to blob, etc.
