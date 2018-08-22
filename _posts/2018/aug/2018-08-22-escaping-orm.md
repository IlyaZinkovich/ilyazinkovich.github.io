---
layout: post
date: 2018-08-22
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

The central domain concept in our system is a **ticket**. **Tickets** are reported by **colleagues** collaborating on a project and contain **description**, **reporter name**, and a list of **comments** with some **text** and the **author**.

Experienced enterprise Java developer quickly builds a relational model of our domain.

![alt text](https://bit.ly/2was1Ll?style=centered "relational tickets")

Then, keeping an eye on [the cheat-sheet](https://en.wikibooks.org/wiki/Java_Persistence/OneToMany), the same developer creates a JPA mapping.

{% gist edb1a74b565dee7853813e245aae12a4 %}

{% gist 4b8ba664d89b17fb7ec38853fa6856d8 %}

{% gist acfe216f415744976768147b45d8a849 %}

Here comes a question: how to create and persist a new comment?  
And we realize there are multiple ways to do this and it's hard to say which one fits better.

{% gist 4254df9747b7b7a5de535594e24b4e0a %}

Which one is going to work in our case depends on the parameters of JPA annotations in our mappings ([on cascade effects in particular](https://vladmihalcea.com/a-beginners-guide-to-jpa-and-hibernate-cascade-types/)).  
Most of the time people fiddle with code until something finally works. Rest assured, you'll see all three approaches together on a real project.  

This traditional mapping creates direct coupling between all the entities. The inherent complexity of objects persistence affects the rest of the application leaking into the business logic. The problem gets even worse over time until the codebase becomes one huge mess.  
However, there is a solution.

## Modular Approach

Let's take the database schema above and think of a better way to map our domain objects to the database.

`@OneToMany` and `@ManyToOne` JPA annotations seem as they are the only means to represent the one-to-many relationship. That's not true and here is an alternative:  

{% gist 2e9dfb6d034f559d6527397b5a331f2e %}

{% gist d0a49ec804ed9f950508a3a452aa609d %}

{% gist 2f0fa39466d1a6fef7cfb46b269f3885 %}

These objects do not contain a direct reference to each other. There are no collections and no JPA annotations. Instead, these objects resemble the database tables referencing each other by id. The mapping becomes very simple. 

{% gist f133473bf775bfe7af6254dad63354fb %}

Yes, this is XML configuration. Although I'm not a huge fan of XML, there is no other way to keep your domain objects clean and at the same time provide persistence support with Hibernate. However, I'd even argue that it gives better visibility of the mapping than the annotation-based approach. If you don't like it, you can use JPA annotations.  
Another thing that may seem strange is that all identifiers have wrappers. `authorId` is not just a `long` value - it's an instance of `AuthorId` class. It helps to make your domain logic free of implementation details such as the internal representation of identifiers and provides a meaningful abstraction for dependency between the objects in your system instead of direct object references.  

Let's persist a comment with a new mapping.

{% gist 323014e270c79a0fa4d2c56b139a8ea9 %}

Notice how simple it is. Now, there is only one way to persist the comment, and it doesn't require the in-depth knowledge of Hibernate framework. Moreover, this approach saves up to two database roundtrips that were previously needed to get the corresponding ticket and author objects for new comment persistence.  
And, most importantly, we paved the way for modularity in our system. 
By default, each entity forms a foundation of a separate module. On the long run, it's much easier to merge independent modules than decomposing already coupled mess. If we see a natural dependency between entities required to keep our model consistent, we can represent this dependency in code, but not the other way around.  

We can add any functionality to existing modules and introduce new modules without worrying that they will affect or complicate each other. We can assemble the following view with asynchronous/parallel calls to respective modules keeping our system resilient to individual module failure. 

![alt text](https://bit.ly/2whmX7Z?style=centered "all modules")

Operations such as status change, colleague avatar change, commenting or time tracking are local to the module. It means we can implement and tune them independently.  

## Data Collocation

Real-life requirements might be more complicated.  

Suppose we need to show **the comments together with their author name** on the same view.  

It might be tempting to use the traditional mapping in which comment object already has a reference to its author. The following code together with eagerly loaded authors will produce the required result.

{% gist d76e82628aa3159ce9b4faa0fda1793b %}

However, physical collocation of data from multiple modules doesn't automatically mean that these modules must have coupling.
The following code retrieves comments and authors data for each comment with just two calls to respective modules.

{% gist 8d837f48f67f8f4b35c10fd874ffba0d %}

As you may notice, our modular mapping is highly optimized for writes, but reads might lead to higher complexity.
Sometimes it's just not enough to have one model for both reads and writes. Writes are supposed to operate as little data as possible in the simplest possible way. Reads are completely different. They aggregate data from multiple sources and provide advanced querying capabilities such as filtering, pattern matching, pagination. 
The solution is to create a separate model that fits domain use cases for reads ([CQRS](https://martinfowler.com/bliki/CQRS.html)). It might be as simple as just providing a set of objects that query the database using raw SQL or as complex as synchronizing our data with a full-text search engine.  
On a large scale, it might be worth creating a read model inside `comments` module that asynchronously gets comments-related data about colleagues such as their names and avatars in the form of events and combines this data with comments.

![alt text](https://bit.ly/2La204p?style=centered "comments read model")

## Cross-Module Requirements

Here comes a new requirement. When ticket status is updated to "done" set remaining time to 0.

Traditionally we'd implement this requirement creating a dependency between ticket management and time tracking functionality.

{% gist 57769914c84d228e407e39002ac4cd16 %}

Imagine how this code grows when we introduce new effects of ticket status update. Each effect adds complexity and requires the client to think about potential failures in effect functionality. Do we need to revert the ticket status update if the remaining time cannot be updated right now? 

The whole time management functionality can be offline, and the client won't even see the inconsistency in a system. We can accept updating the remaining time in the background. But how to achieve it?  
With the modular design, we can leverage the publish-subscribe mechanism.

![alt text](https://bit.ly/2PkOjmm?style=centered "publish-subscribe")

`ticket` module publishes events for each status change, while `time tracker` processes them in the time tracking context. 

{% gist ecb7950320085366d4096a66325b3760 %}

This simple solution naturally evolves into a scalable and resilient distributed system with services asynchronously communicating via messaging middleware.  

## Summary

Finally, it appears that the ORM is not the main problem. You can still use this tool while keeping an eye on the system modularity. And modularity is a key that enables agility. With a reduced complexity of our system components and a clearly defined contract between them, we can build better software that serves its purpose and aims towards the future.
