---
layout: post
date: 2018-06-09
title: "Data-Driven Changes in Distributed Systems"
description: |
keywords:
  - scientist
  - redesign
  - distributed systems
categories: distributed systems
urlimage: https://bit.ly/2Hzbuo2
---

When we make changes that span across multiple services in a distributed system we face series of challenges. Risk of breaking the whole system is very high.  

> Integrating two software systems is usually more like performing a heart transplant than snapping together LEGO blocks.  
> -- John D. Cook, <cite>[LEGO Blocks and Organ Transplants, 3 February 2011](https://www.johndcook.com/blog/2011/02/03/lego-blocks-and-organ-transplants/).  

This is very unfortunate, but we need to adapt to these obstacles and develop better tools and practices to overcome these problems.  
GitHub developed a small library for fearless refactoring called [scientist](https://github.com/github/scientist). The idea is to make data-driven decisions on refactored code rollout instead of pushing it into production and hoping that nothing will crash (while keeping in mind a rollback).  
I’d like to extend this idea to cover not only refactoring but also the system level changes.

<!--more-->

## Data-Driven Refactoring

When you refactor the code in a mission-critical part of the business flow, it's generally a good idea to keep non-refactored code along with refactored one. Then you can use [feature toggles](https://martinfowler.com/articles/feature-toggles.html) to rollout refactored code to production gradually.

{% gist a681c9789c26078a19fb8ed5243e2332 %}

This approach is viable, but it has some flaws. How quickly can we switch back to the old solution? Will it be safe? What if the cost of failure is very high? What will happen if we switch this year-old toggle?  
We need to address all these questions. Most of the time it can lead to much more complicated rollout strategies and high risks.  

However, there is another approach. Instead of making the switches between different code paths we can run them in parallel and compare their results.  

{% gist 7ead384ba0bf547af0bb54932141a39c %}

Results of experiments are collected and published to the metrics visualization tool of choice where you can observe them for some period of time. Finally, you can make a data-driven decision to rollout refactored code and remove the old one.  

This practice seems very useful if the new and old code paths **don't modify any shared state**.

## System Level Changes

Recently we discussed [designing reliable service communication](https://ilyazinkovich.github.io/2018/06/02/reliable-services-communication.html). Let's use the GitHub's approach to migrate from synchronous services communication to asynchronous one.  

Quick reminder, we're building a delivery service in which we pay couriers per delivery. Payments service controls payment process, Incentives service - updates to the couriers' performance. Payments service communicates with Incentives to make a payment adjustment based on the courier performance. And we decided to replace synchronous communication between these two services with asynchronous.

![alt text](https://bit.ly/2Hz7bJo?style=centered "redesign plan")

Using non-invasive Change Data Capture technique (better described in a lightening book [Designing Data-Intensive Applications](https://amzn.to/2xB9lVv)) we make Incentives service publish performance updates in the form of events. On the other side, we add an event consumer that populates performance cache for Payments service. Both event producer and consumer shouldn't necessarily be physical parts of our services. We can even implement them using [AWS Lambda](https://aws.amazon.com/lambda).  

![alt text](https://bit.ly/2kXMJsD?style=centered "performance updates pipeline")

Now let's zoom into the payment process.  
We're not sure if this asynchronous communication will work for us. That's why we set up an experiment. The control function will make a call to Incentives service as usual, while the candidate function will read performance data from the cache. We compare the values returned by both functions and log the result of the experiment.  

![alt text](https://bit.ly/2Hzbuo2?style=centered "experiment")

I've built a [small library](https://github.com/IlyaZinkovich/scientist) to express this in Java code. This is how it looks now:

{% gist 275944673a914508e779e4942fd47886 %}

And this is what happens under the hood:

{% gist da4b479fa7e0e067e5a00c91247e5bdc %}

Experiment invokes candidate function asynchronously without interrupting the main flow.
Collected experiment results are exported periodically in the following format:

{% gist 23088f4591f94735f3838bb7fdff2331 %}

Looks like our solution is pretty stable. Now it's time to remove the experiment and submit the redesigned code.

Here are the links to repos containing source code: 
[https://github.com/IlyaZinkovich/data-driven-changes](https://github.com/IlyaZinkovich/data-driven-changes)
[https://github.com/IlyaZinkovich/scientist](https://github.com/IlyaZinkovich/scientist)
