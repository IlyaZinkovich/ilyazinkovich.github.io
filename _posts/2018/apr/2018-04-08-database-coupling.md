---
layout: post
date: 2018-04-08
title: Uncovering the Database Coupling in Distributed Systems
description: |
keywords:
  - distributed systems
  - database coupling
  - tools
categories: system design
---

In the perfect world, services in distributed systems hide their data behind an interface. 
Although [the shared database](http://www.enterpriseintegrationpatterns.com/patterns/messaging/SharedDataBaseIntegration.html) is a well-known services integration [anti-pattern](http://www.ben-morris.com/a-shared-database-is-still-an-anti-pattern-no-matter-what-the-justification), it's still one of the most used ones due to its reduced initial complexity and promising consistency guarantees.  

Extracting services in a system with an overused shared database integration style becomes a nightmare. Even if you find a sub-domain to isolate, you still need to decouple its data from the rest of the system.  

My point is that we can gradually solve this problem avoiding massive [rewrites](https://www.youtube.com/watch?v=AXU4-VlAAcg). But we need better tools to do this.

<!--more-->

Inspired by the [Structure101](https://structure101.com) project and lessons learned while developing a [coupling analysis plugin](https://github.com/IlyaZinkovich/coupling-analysis), I'd like to share my thoughts on what we need to measure and deal with coupling in distributed systems.

Imagine a situation when you have a core service in your system and a recently extracted service accessing the same data store. The core interacts with a new service to fulfill its business logic objectives but still has access to its underlying data.

![alt text](https://bit.ly/2qeesaM?style=centered "system under dicsussion")

This setup may lead to unexpected behavior in a newly extracted service that may (and usually will) rely on the full control over its data.  
The idea is to identify potential pain points and deal with them on a case-by-case basis.  

Tools needed:
- [database access logging](https://vladmihalcea.com/the-best-way-to-log-jdbc-statements)
- [distributed tracing](https://cloud.spring.io/spring-cloud-sleuth)
- [graph database](https://neo4j.com)

Combining this all together, I've created a [project](https://github.com/IlyaZinkovich/systems-coupling-analysis) that consists of three parts:
- plugin that seamlessly integrates with your SpringBoot app and logs API requests and database access.
- app that analyses the logs and stores them in the graph database.
- simulation of the system discussed here.

Simulation analysis produces a graph stored in Neo4j.
Querying the graph via the web interface uncovers the following structure.

![alt text](https://bit.ly/2HjiwhY?style=centered "whole system graph")

The mobile client accesses services API to create some records in the database and get them later. Core and sub-domain services access the table in the underlying data storage using SQL queries. But there is something more.

Nailing down the problem with another query, we identify the data leak.

![alt text](https://bit.ly/2EtAz2b?style=centered "data leak")

When the mobile client accesses the core API, the core gets some data from the sub-domain service and additionally retrieves some data from the database table directly.

Problem identified. The only thing left is to grab your software skills and fix it.
