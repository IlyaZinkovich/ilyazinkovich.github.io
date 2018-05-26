---
layout: post
date: 2018-05-27
title: "Consistency in Event Sourced Systems"
description: |
keywords:
  - event sourcing
  - consistency
  - actors
categories: event sourcing
---

Event Sourcing recently became very popular with the rise of [reactive systems](https://www.reactivemanifesto.org/). However, the idea is not new and has a very concise definition polished with the years of research:  
>"Capture all changes to an application state as a sequence of events"  
>[Martin Fowler, 12 December 2005](https://martinfowler.com/eaaDev/EventSourcing.html).  

Although event sourcing is not directly related to eventual consistency (apart from the word "event" in the name), the common misconception is that you get eventual consistency for free while building event sourced system.  
In reality, the only kind of consistency you get for free is **in**consistency.  
Let's explore how not to get into this trap and build an event sourced system providing strong consistency guarantees.

<!--more-->

### Introducing the Shopping Cart Example
Everything starts with modelling the problem domain.  
Our domain will be e-commerce where we'll model a popular shopping cart concept.  
We can add products to the shopping cart until we reach its capacity and query the quantity of products in it.

![alt text](https://bit.ly/2IJ2gea?style=centered "domain model")

### Promising Event Sourced Shopping Cart Implementation
When you are first getting familiar with event sourcing you most likely build the following kind of a system.

{% gist 678b251592d522bf2883a51f7a17351b %}

In order to fix this unfortunate issue we need to redesign our solution.
Keeping in mind [the first law of distributed systems](https://martinfowler.com/bliki/FirstLaw.html), let's build a system where there is only one instance of each aggregate at runtime.

In reality, when you're building a scalable e-commerce website, you will create a `ShoppingCart` actor per user and shard these actors multiple machines using [akka-cluster](https://doc.akka.io/docs/akka/2.5/cluster-usage.html) while preserving the essential constraint of having only one instance of aggregate at runtime. Then you can enrich your system with eventually consistent projections of the event store.

And still the `ShoppingCart` aggregate will be a small island of strong consistency in the cruel world of distributed systems.

Keep an eye on your distributed system, define consistency boundaries right and don't follow the trends blindly.
