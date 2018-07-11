---
layout: post
date: 2018-07-17
title: "Prototyping Distributed Systems with RxJava"
description: |
keywords:
  - prototyping
  - distributed systems
  - rxjava
categories: distributed systems
urlimage: 
---

When we design a distributed system often comes a moment when we agree on the high-level architecture, but we're still uncertain that it fits the business process we're modeling. The most common way to overcome the dead end is to start building a full-blown solution and calibrate it along the way.  
Alternatively, I propose to use lightweight prototyping techniques. Representing distributed system design in a runnable code, we can enable experimentation and reduce risks associated with bad architectural decisions as early as possible.  
Let's prototype a part of a large-scale delivery service using building blocks from RxJava.

<!--more-->

## Large-Scale Delivery Service

The fundamental part of our delivery service is a dispatch of orders to the couriers.  
The business process consists of a few steps: 
1. search couriers in the order pick-up area;
2. filter couriers according to business rules;
3. sort remaining couriers by estimated time of arrival (ETA) 
assigning the courier with the least ETA to the order.

The following sketch defines fundamental components of our initial design.
![alt text](https://bit.ly/2NVG3sv?style=centered "dispatch flow")

The system should sustain a large flow of incoming orders and support evolution of each stage in the process.
We decided to build an event-stream processing pipeline matching the business process.

## Prototyping

We can model each step of the process with a regular Java object accepting incoming messages and producing computation results. Hopefully, Java 8 comes with a built-in [Consumer](https://docs.oracle.com/javase/8/docs/api/java/util/function/Consumer.html) abstraction that we can use to decouple a message producer from a message consumer. Using this primitive, we can implement the dispatch process components as follows:

{% gist e08ba51b1aecb8d45a8895e4f04d3ebd %}
{% gist c64cd11514b75e9b74d7014d6bca4dbb %}
{% gist 1d9d447c168275385e75e5d02622b3bf %}

In a production system, these components would likely be tied together using some messaging technology like [Kafka](http://kafka.apache.org/), [Kinesis](https://aws.amazon.com/kinesis/data-streams/) or [RabbitMQ](https://www.rabbitmq.com/).  
Instead, in our prototype, we can use a lightweight implementation of [the publish-subscribe pattern](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern) from [RxJava](https://github.com/ReactiveX/RxJava). RxJava is a reactive programming library built around the concept of observable data streams. We'll use [PublishSubject](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/subjects/PublishSubject.html) primitive from this library to model publish-subscribe communication style between system components.  

The following test combines all components together:

{% gist 09f179c250f8ff1b4620da3f4b79e179 %}

This test passes under the following optimistic conditions:
1. Search finds at least one courier per order pick-up location.
2. Filter leaves at least one courier unfiltered.  

But the reality is different. There might be no couriers in a specified location suitable for the order.  

It's time to ask domain expert what should happen in this situation. 
One possible answer is to repeat the dispatch process a few times and return error response if we cannot fulfill the order.  
Search and Filter components share retry logic. It might be a sign that we need to extract it into a separate component. Let's try it out.

The high-level diagram will look a bit different now:

![alt text](https://bit.ly/2NmQhB3?style=centered "dispatch flow")

Redispatch component won't differ too much from the others:

{% gist 71da610f78cfc720497dbd8945b7766f %}

And the final test will look as follows:

{% gist 2c21c5be16fb6915ff18d000d0df8c1f %}

We've spent about an hour implementing this prototype and already discovered a new component in our system. And we've done it when the cost of change is very low - before we even started implementing the system.  
Moreover, we've built a platform for experimentation. If anyone questions the architecture, we can write a unit test to prove if the argument is valid.  
From time to time, this kind of prototypes can even be used as the first iteration of a real system. 

Now nothing stops you from replacing RxJava with Kafka for better durability and use thread pools to leverage all compute resources. However, these are just implementation details.

Here is a link to the repository containing source code: 
[https://github.com/IlyaZinkovich/reactive-delivery](https://github.com/IlyaZinkovich/reactive-delivery)
