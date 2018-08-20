---
layout: post
date: 2018-08-26
title: "Escaping the ORM-Imposed Coupling"
description: |
keywords:
  - orm
  - design
  - coupling
categories: design
urlimage: 
---

Object-Relational Mapping ([ORM](https://en.wikipedia.org/wiki/Object-relational_mapping)) is one of the most controversial but still widely used tools in web application development. Ted Neward colorfully explained the most fundamental problems with ORM back in 2006, calling it The Vietnam of Computer Science.

>"In the case of automated Object-Relational Mapping, early successes yield a commitment to use ORM in places where success becomes more elusive, and over time, isn’t a success at all due to the overhead of time and energy required to support it through all possible use-cases."  
>[Ted Neward, 26 June 2006](http://blogs.tedneward.com/post/the-vietnam-of-computer-science/)

Despite all the blame, ORM appeared on all the projects I worked on so far.
Unfortunately, most of them had unnecessary complexity due to the traditional use of the ORM.  
This time I'd like to discuss the roots of this problem and provide a solution to reduce the complexity and bring better modularity to your software system.

<!--more-->

## Traditional Approach

Suppose we're building an issue-tracking system like JIRA and it's going to be a Java application with data stored in a relational database.

The central domain concept in our system is a **ticket**. Tickets are reported by colleagues collaborating on a project and contain **description**, **reporter name**, and a list of **comments** with some **text** and the **author**.

Experienced enterprise Java developer quickly builds a relational model of our domain.

![alt text](https://bit.ly/2was1Ll?style=centered "relational tickets")

Then, keeping an eye on [a cheat-sheet](https://en.wikibooks.org/wiki/Java_Persistence/OneToMany), the same developer creates a JPA mapping.

{% gist edb1a74b565dee7853813e245aae12a4 %}

{% gist 4b8ba664d89b17fb7ec38853fa6856d8 %}

{% gist acfe216f415744976768147b45d8a849 %}

Here comes a question: how to create and persist a new comment?  
And we realize there are multiple ways to do this and it's hard to say which one fits better.

{% gist 4254df9747b7b7a5de535594e24b4e0a %}

Which one is going to work in our case depends on the parameters of JPA annotations in our mappings.  
Most of the time people fiddle with code until something finally works. Rest assured, you'll see all three approaches together on a real project.  

This traditional mapping creates direct coupling between all the entities. The inherent complexity of objects persistence affects the rest of the application leaking into the business logic. The problem gets even worse over time until the codebase becomes a huge mess.  
However, there is a solution.

## Modular Approach

Suppose we're already in trouble. We inherited the legacy system above, and we're not allowed to change the database or even the database schema. However, we can still change the mapping.

`@OneToMany` and `@ManyToOne` JPA annotations seem as they are the only means to represent the one-to-many relationship. That's not true and here is an alternative:  

{% gist 2e9dfb6d034f559d6527397b5a331f2e %}

{% gist d0a49ec804ed9f950508a3a452aa609d %}

{% gist 2f0fa39466d1a6fef7cfb46b269f3885 %}

These objects do not contain a direct reference to each other. There are no collections and no JPA annotations. Instead, these objects resemble the database tables referencing each other by id. The mapping becomes very simple. 

{% gist f133473bf775bfe7af6254dad63354fb %}

Yes, this is XML configuration. Although I'm not a huge fan of XML, there is no other way to keep your domain objects clean and at the same time provide persistence support with Hibernate. However, I'd even argue that it gives better visibility of the mapping than the annotation-based approach.  
Another thing that may seem strange is that all identifiers have wrappers. `authorId` is not just a `long` value - it's an instance of `AuthorId` class. It helps to make your domain logic free of implementation details such as the internal representation of identifiers and provides a meaningful abstraction for dependency between the objects in your system instead of direct object references.  

Let's persist a comment with a new mapping.

{% gist 323014e270c79a0fa4d2c56b139a8ea9 %}

Notice how simple it is. Interestingly, now there is only one way to persist the comment, and it doesn't require the in-depth knowledge of Hibernate framework. Moreover, this approach saves up to two database roundtrips that were previously needed to get the corresponding ticket and author objects. There is no direct coupling between objects in our system anymore.  
Notice how we use the term `Author` that better fits our comments domain than the general `Colleague` term, although the corresponding table still has `COLLEAGUES` name. This naming technique helps to bridge the gap between domain experts and developers who implement the system with the shared domain language.

However, not all business requirements are easy to implement in a new model. Suppose we need to show the comments together with their authors on the same view.  
It's easy to do with the traditional model.

{% gist d76e82628aa3159ce9b4faa0fda1793b %}

Alternative will look as follows.

{% gist 8d837f48f67f8f4b35c10fd874ffba0d %}

This complexity might not be acceptable. But this is not a problem of our modular mapping in particular.  
Sometimes it's just not enough to have one model for both reads and writes. Writes are supposed to operate as little data as possible in the simplest possible way. Reads are completely different. They aggregate data from multiple sources and provide advanced querying capabilities such as filtering, pattern matching, pagination. 
The solution is to create a separate model that fits domain use cases for reads ([CQRS](https://martinfowler.com/bliki/CQRS.html)). It might be as simple as just providing a set of objects that query the database using raw SQL or not as simple as synchronizing our data with a full-text search engine.  
I remember one project on which we needed to provide full-text search capabilities for the data stored in the Oracle database. We decided to use Hibernate Search on top of traditional JPA mapping. It was a dead-end. Today I would seriously consider having a separate model.

## Conclusion

I'd advise careful research of the read and write patterns in your software system. General solutions recommended in tutorials don't fit all the problems. Lean towards the simplest, consistent and modular model and keep in mind that there might be more than one model to solve the problem in the most optimal way.
