---
layout: post
date: 2018-05-26
title: "Consistency in Event Sourced Systems"
description: |
keywords:
  - event sourcing
  - consistency
  - actors
categories: event sourcing
---

Event Sourcing became very popular with the recent rise of [reactive systems](https://www.reactivemanifesto.org/). However, the idea is not new and has a very concise definition polished with the years of research:  
>"Capture all changes to an application state as a sequence of events"  
>[Martin Fowler, 12 December 2005](https://martinfowler.com/eaaDev/EventSourcing.html).  

Although event sourcing is not directly related to eventual consistency (apart from the word "event" in the name), the common misconception is that you get eventual consistency for free while building event sourced system.  
In reality, the only kind of consistency you get for free is **in**consistency.  
Let's explore how not to get into this trap and build an event sourced system providing strong consistency guarantees.

<!--more-->

## Introducing the Shopping Cart Example
Everything starts with modeling the problem domain.  
Our domain will be e-commerce where we'll model a popular shopping cart concept.  
We can add products to the shopping cart until we reach its capacity and query the number of products in it.

![alt text](https://bit.ly/2IJ2gea?style=centered "domain model")

## Promising Event Sourced Shopping Cart Implementation
When you are first getting familiar with event sourcing you most likely build the following kind of a system.

{% gist 678b251592d522bf2883a51f7a17351b %}

This design is good enough for a single-threaded environment on a single server. But adding this class to the concurrent environment like a Spring Boot web application can cause havoc that can be simulated with the following test.

{% gist e02b7d13585bff73461e9911cd7621d2 %}

In each thread, we create a `ShoppingCart` instance from the event store and send multiple `AddProduct` commands.
Then we verify that the shopping cart is full and the main business rule is not violated: "number of products in a cart is not greater than the cart capacity". Each time we get a different number of products in a shopping cart.
This code is obviously not thread-safe and doesn't scale out no matter what technology you use for event store (Kafka, Cassandra, Dynamo DB, you name it). Although it's a legit event sourcing implementation, the application state quickly becomes inconsistent instead of being eventually consistent as we thought.

## Last Hope

In order to fix this unfortunate issue, we need to redesign our solution.
Keeping in mind [the first law of distributed systems](https://martinfowler.com/bliki/FirstLaw.html), let's build a system where there is only one instance of each aggregate at runtime.
Instead of reinventing the wheel, we'll use high-level concurrency primitives from [akka](https://akka.io/), building our system using actor model.

{% gist 9e831563163646ea29fa3bd07f85414f %}

The the rest of the application code should ensure that we create the instance of this actor only once and then reference it by its name.

{% gist d4e366d945a48dfda4eba3f2c6b9cb64 %}

The following test proves the strong consistency of this solution.

{% gist 620f977331d4c7eb7bf94cbd0e743d89 %}

While operating on a large scale, you'll create a `ShoppingCart` actor per client of your app and shard these actors to multiple machines using [akka-cluster"](https://doc.akka.io/docs/akka/2.5/cluster-usage.html) while preserving the essential constraint of having only one instance of aggregate at runtime. Then you can enrich your system with eventually consistent projections of the event store.

And still, the `ShoppingCart` aggregate will be a small island of strong consistency in the cruel world of distributed systems.

Here is a link to the repository containing source code: [https://github.com/IlyaZinkovich/event-sourcing-consistency](https://github.com/IlyaZinkovich/event-sourcing-consistency)
