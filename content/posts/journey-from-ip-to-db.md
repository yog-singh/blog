---
title: Journey from IP packet to DB row
author: Yogesh Singh
date: 2022-02-19 11:00:00 +0530
categories: [blog, Database]
tags: [postgres, database]
---
<br>
Databases are an integral part of today's society. Even without realising we use databases in our day-to-day activities, while making online transactions, shopping our favorite cookies et cetera. As developers we interact with databases for storing data of various kinds.

This post walks though the birds eye view of how database engines function and work to store and read data. We will be focusing on *Postgres* as database and *Java* as the application language to interact with the DB.

## Source of data/query
The data generally originates in any backend application, here let's consider a Java application. Data received from user is generally mapped to a Java object and thereon the object is mapped to a database relation using the Object Relation Mapping (ORM).

ORM does the major task of mapping objects to the relation/table and creating queries for operations requested by the application and handles all the interaction with database.

Postgres client library is the interface between the backend application and the database. The ORM interacts to the DB using the client library. PostgreSQL JDBC Driver is the client for Java. JDBC maintains the connection pool and sends over all the query requests to Postgres.

So basically ORM converts every save and get request to respective SQL queries which  is handed over to Postgres in plain text.

## Path of a Query
Postgres client library sends the queries in the dialect which Postgres understands and also receives the results which is handed over to the ORM for further processing.

Now when the SQL query reaches the postgres server, the clock starts ticking and the query needs to be executed as fast as possible and the result set returned to the client. The process of query execution is divided into multiple parts as described in the diagram below:


### Parser

The first stage of execution begins at the parser when the query is received. The task of parser is to lexically analyze the query for syntactical errors and then parse the query to generate the parse tree.

Parser further consists of the lexer and parse engine. Lexer used by Postgres is defined in the file *scan.l* and recognizes identifiers, the SQL keywords etc. The SQL query is tokenized and if any syntax error is found in the query error is generated.

The tokenized query is handed over to the parser for generating the parse tree. The parser being defined in file *gram.y* and this consists of the grammar rules and actions that are executed whenever a rule is fired. The code of the actions (which is actually C code) is used to build up the parse tree.

Using the syntactical structures of SQL the parser stage creates the *parse tree*. After this process the transformation process takes the tree handed back by the parser as input and does the semantic interpretation needed to understand which tables, functions, and operators are referenced by the query. The data structure that is built to represent this information is called the *query tree*.

### Optimizer

The *optimizer/planner* is handed over the task of creating an optimal execution plan. A given SQL query (and hence, a query tree) can be actually executed in a wide variety of different ways, each of which will produce the same set of results. If it is computationally feasible, the query optimizer will examine each of these possible execution plans, ultimately selecting the execution plan that is expected to run the fastest.

The planner/optimizer starts by generating plans for scanning each individual relation (table) used in the query. The possible plans are determined by the available indexes on each relation. If the query requires joining two or more relations, plans for joining relations are considered after all feasible plans have been found for scanning single relations. When the query involves more than two relations, the final result must be built up by a tree of join steps, each with two inputs. The planner examines different possible join sequences to find the cheapest one.
If the query uses fewer than *geqo_threshold* relations, a near-exhaustive search is conducted to find the best join sequence. If the joins exceed the threshold then Postgres uses `Genetic Query Optimizer` for determining a reasonable query plan in a reasonable amount of time.

### Executor

Once the query tree and plan is ready, it is handed over to the executor to fetch the result. The executor takes the plan created by the planner/optimizer and recursively processes it to extract the required set of rows. This is essentially a demand-pull pipeline mechanism. Each time a plan node is called, it must deliver one more row, or report that it is done delivering rows. The result set generated is returned to the client.

So that's how every CRUD action in all the backend applications are sent over and processed by Postgres. In the next post I would be covering how Postgres stores all data on the physical storage.
