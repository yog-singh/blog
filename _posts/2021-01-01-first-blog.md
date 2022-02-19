---
title: First Blog
author: Yogesh Singh
date: 2021-01-02 11:00:00 +0530
categories: []
tags: [postgres, database]
---
# Journey from IP packet to DB row
<br>
Databases are an integral part of today's society. Even without realising we use databases in our day-to-day activities, while making online transactions, shopping our favorite cookies et cetera. As developers we interact with databases for storing data of various kinds.
<br>
This post walks though the birds eye view of how database engines function and work to store and read data. We will be focusing on *Postgres* as database and *Java* as the application language to interact with the DB.

## Source of data
The data generally originates in any backend application, here let's consider a Java application. Data entered by user is generally mapped to a Java object and thereon the object is mapped to database relation using the Object Relation Mapping (ORM).

![Java Object to ORM Object] ({{ site.baseurl }}/blog/assets/img/diagrams/post-1-java-to-orm-object.png)
