---
layout: post
date: 2019-01-18
title: Separating Dynamic Configuration from Business Logic
description: |
keywords:
  - modular design
  - configuration
categories: modular design
urlimage: 
published: false
---

Modern businesses demand higher flexibility of our applications.  
Emerging data-driven approaches to product development encourage A/B testing to gradually improve product market fit.  
Continuous Delivery popularises decoupling of feature releases from deployment cycle.  
These techniques require some sort of dynamic configuration of our applications. Poorly implemented, the dynamic configuration makes the code fragile. Being not able to change the software frequently we loose all the benefits of delivering continuously and being data-driven. In this article I'm presenting an approach to control the dynamic configuration.

<!--more-->

## Common Approach to Dynamic Configuration

[Feature Toggles](https://martinfowler.com/articles/feature-toggles.html) in some form or another is a common approach to dynamic configuration.

You can recognise this kind of code that is spoiled with A/B-testing and feature-toggle library.
The code quickly becomes full of various in-place switches that are very simple to implement and are supposed to last no longer than the next deployment or A/B-testing period. However, as practice shows, we often encounter years-old toggles or a switch for an A/B-test that started months ago and the team hardly knows if it's still in progress.
This assumption might be wrong for other reasons.
Have you been in a situation when as a result of A/B test business decides to have both versions (A and B) but that should be applied to different user groups dynamically?

This approach has its roots in the Working Effectively with Legacy Code book. There is a concept of a seam that allows to switch behaviour of dependent module in prod and testing environments.
But what if we want to switch this behaviour dynamically in a production system based on some signal or for different client groups.

Let's say we're developing a car-sharing service and need to get cars nearby client location filtered according to his preferences. We have a couple of filters:
- prodFilter that filters cars according to the last stable version of a filter;  
- candidateFilter that was developed today and should be switched on and off with a feature toggle and finally replace the prodFilter;  
- awesomeFilter which is an experimental filter that targets a specific client group according to A/B test configuration based on the client id.

{% gist e29341606c2106bb4871bc6bdf938794 %}

This is an oversimplified version of what you might find in production code. You can imagine how the size of each if-else-statement blocks grows and it becomes unclear where to put the new business logic. 
Using the feature toggles among the business logic code ties us closer to the feature toggle implementation. Changing this becomes very risky. 
Some of the feature toggle libraries make blocking remote calls to determine the state of a toggle. This is a huge issue since all of our domain tests should include some way of mocking the feature toggles.
Let's modularise this code and extract the dynamic configuration part.

## Extracting Configuration from Business Logic

{% gist 8f95e2f41c0af1afefd9fce6eb530d79 %}

Since the domain code no longer depends on the configuration, this approach has multiple benefits:
- business logic can be easily tested separately from the configuration code and requires no mocking whatsoever;
- all configuration is in one place outside of the business logic, which gives us a better view on the way our application is configured and clear actionable feedback if the number of configurations grows too much;
- we can easily change the approach to dynamic configuration without a huge risk of breaking the domain code;

Imagine that after the A/B test we decided to apply different filters for different client groups.
Then we can implement AwesomeFilter the following way:

{% gist 348fc7780a934142b43486d4e9939cfe %}

AwesomeFilter becomes a part of our business and its configuration is still placed outside.

If you constantly apply this technique, you can find yourself building more modular system that can be composed in multiple ways based on the cutomer needs. It encourages creation of smaller modules with clearly defined interfaces that can be switched on and off, optimised and rewritten.
