+++
title = "Primer in databases"
date = 2024-08-09
updated = 2024-08-09
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

It is important to note that only 2 of these 3 can be provided at one time.

1. Consistency: For each read we read the latest data written into the database. 

Let's see when I think it is a great idea to use single-instance databases:

- Local application states: The settings of an application, game states, etc. Mostly done via SQLite. As this data is not critical and only accessed locally speed is the upmost importance here. If there is any inconsistency it can be easily fixed, but due to the simple nature of it there is no real danger of inconsistencies. Maybe a race condition or a critical system error can trigger inconsistent writes, but that is only from 1 source anyway. Performance is extremely important for more complex databse system, for instance in games game assets need to be loaded on the fly in a smart way.
