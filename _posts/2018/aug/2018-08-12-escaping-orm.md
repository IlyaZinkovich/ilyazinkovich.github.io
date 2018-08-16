---
layout: post
date: 2018-08-30
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
This time I'd like to discuss the roots of this problem and provide a solution to reduce the complexity and bring better modularity to your system without much effort.

<!--more-->

## Traditional Approach

Suppose we're building an issue-tracking system like JIRA and it's going to be a Java web application with data stored in a relational database.

The central domain concept in our system is a **ticket**. Tickets are reported by colleagues collaborating on a project and contain **description**, **reporter name**, and a list of **comments** with some **text** and its **author**.

Experienced enterprise Java developer quickly builds a relational model of our domain.

![alt text](https://bit.ly/2was1Ll?style=centered "relational tickets")

Then, keeping an eye on [a cheat-sheet](https://en.wikibooks.org/wiki/Java_Persistence/OneToMany), the same developer creates a JPA mapping.

{% gist edb1a74b565dee7853813e245aae12a4 %}

{% gist 4b8ba664d89b17fb7ec38853fa6856d8 %}

{% gist acfe216f415744976768147b45d8a849 %}

Here comes a simple question: how to create and persist a new comment?  
And we realize there are multiple ways to do this and it's hard to say which one fits better.

{% gist 4254df9747b7b7a5de535594e24b4e0a %}

Which one is going to work in our case depends on the parameters of JPA annotations in our mappings.  
Most of the time people fiddle with code until something finally works. Rest assured, you'll see all three approaches together on a real project.  

This traditional mapping creates direct coupling between all the entities. The inherent complexity of objects persistence affects the rest of the application leaking into the business logic. The problem gets even worse over time until the codebase becomes a huge mess.  
However, there is a solution.

## Modular Approach

Suppose we're already in trouble. We inherited the legacy system above, and we're not allowed to change the database or even the database schema.

`@OneToMany` and `@ManyToOne` JPA annotations seem as they are the only means to represent the one-to-many relationship. That's not true. 

{% gist 2e9dfb6d034f559d6527397b5a331f2e %}

{% gist d0a49ec804ed9f950508a3a452aa609d %}

{% gist 2f0fa39466d1a6fef7cfb46b269f3885 %}
<!-- It's a simple problem without a precise answer, but it's not the only one.

1. Should we use lazy or eager loading?  
2. When should we intialize collection representing one-to-many relation? Which type of collection to use?  
3. When should we stop initializing the object graph if it has cycles (ticket -> reporter -> comment -> ticket)?   -->

Did you ever ask yourself why you made a decision to map objects to database this way?
One possible answer is that business requirements are so complex that you need an ability to traverse the whole objects graph back and forth to implement sophisticated business rules. 
For example, the reporter of the ticket cannot write more than 5 comments under the reported ticket.

However, most of the time this is not the case. 
