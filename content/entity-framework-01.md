+++
title = "Primer in databases: monolithic or distributed?"
date = 2024-08-09
updated = 2024-08-12
description = ""
[taxonomies]
tags = ["database","SQL","DBMS", ".NET"]
+++

One of the most fundamental pieces of computing is data. Most of tend to think about the way we compute things: the algorithms, the hardware, the networks. But it only works as fast or well as the the data structures behind it.

In practice we use programs which can efficiently store and retrieve on demand our data as we need it.
Or to be more precise....

## The CAP trilemma

To be accurate nowadays databases are not singular, monolithic, even if the end user (an API consumer, a Facebook user, a programmer) might feel like it is.

Traditionally, we run 1 database instance. This is great because it is simple (well, simpler) to configure and run it.

To understand why monolithic or centralized databases are actually a good choice, we must first understand what distributed database systems try to solve. Later on when using databases from C# using the framework components provided by .NET, we will see that most of the implementation details can be left to Entity Framework, but we must have a solid understanding of these ideas.

There are 3 main reasons to consider when using a distributed database:

It is important to note that only 2 of these 3 can be provided at one time fully.

1. Consistency: For each read we read the latest data written into the database.
2. Availability: For each request we get a response, wheter it be a result of successful request or an error.
3. Partition Tolerance: The database works as expected even if some nodes are down or unreachable due to network issues.

These are called together the CAP-trilemma (or some other similar name).
One way to describe a particular type of databse is by mentioning which 2 are fullfilled: CA/CP/AP.

Examples of these groups are:

- AP: Apache Cassandra, CouchDB, DynamoDB.
- CP: Redis, MongoDB, Apache Hbase
- CA: MySQL, Microsoft SQL Server, PostreSQL, Amazon Redshift

Most commonly databases are CA-type.

However, not all cases require anything other than CA. Let's see when I think it is a great idea to use single-instance databases:

- Local application states: The settings of an application, game states, etc. Mostly done via SQLite. As this data is not critical and only accessed locally speed is the upmost importance here. If there is any inconsistency it can be easily fixed, but due to the simple nature of it there is no real danger of inconsistencies. Maybe a race condition or a critical system error can trigger inconsistent writes, but that is only from 1 source anyway. Performance is extremely important for more complex databse system, for instance in games game assets need to be loaded on the fly in a smart way.
- Cache storage: On a server - for instance on a Nextcloud server -  [Redis can be used](https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/caching_configuration.html#id2) as a method to keep track of locked files. As Redis is an in-memory database (well, to be honest Redis can be looked as a key-value storage type as well). I would like to note here that with Redis Clusters you can actually turn Redis from a CA to CP type database.
- Small servers: When working with servers, one should consider size as a determining factor to decide. If it is known that there will be no high load on the server or data is rarely accessed, there is no need to add partitioning to it.
- Development: During active development, it is practical to set up a small local and reproducible deployment of the database. Unless you want to test partitioned instances, it is a lot easier to set up a single partition. Also, for unit tests it is needed in many cases, however it is worth to note that many testing frameworks provide options to add different backends to run the unit tests to decouple possible issues from the database.

I would like to point out that not adding partitioning to the a database is not equivivalent to not making backups. I personally follow the 3-2-1 rule of backups. There should at the very least be 3 copies of the data: 1 instance of the production data, a backup of the production data on a different physical copy and one off-site for disaster recovery. I personally think that the data should be version controlled as well. For me, this means using an incremental backup copy strategy, like Kopia (which is cross-platform and I use for my Nextcloud server and my Linux desktop installtion) and MacOS's included Time Machine.

## Addendum

Today I explained a way of categorizing databases. The reason why I did not start writing about SQL/noSQL, document databases, graph databases is that technically any type of database can be a monolithic or partitioned one and for any and all the CAP-trilemma stands.

Int the next post in this series, I will go into detail in that way, as it will be crucial to understand before writing Entity Framework code.
