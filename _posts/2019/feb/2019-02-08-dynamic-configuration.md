---
layout: post
date: 2019-02-08
title: Isolating Business Logic from Dynamic Configuration
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

[Feature Toggles](https://martinfowler.com/articles/feature-toggles.html) are widely used to configure the applications dynamically. Here is an example of **how** they're used.

Let's say we're developing a car-sharing service and need to get cars nearby client location filtered according to his preferences. We have a couple of filters:
- **prodFilter** that filters cars according to the last stable version of a filter;  
- **candidateFilter** that was developed today and should be switched on and off with a feature toggle and finally replace the prodFilter;  
- **awesomeFilter** which is an experimental filter that targets a specific client group according to A/B test configuration based on the client id.

{% gist e29341606c2106bb4871bc6bdf938794 %}

We're used to this kind of code style. It's easy to write until you cannot recognize what's the business logic of the code hidden behind these nested if-else statements that appear throughout the whole codebase. Ideally, this situation should never happen because we all agree that the toggles are temporary and should be removed as soon as the switching is finished. But as practice shows, stale toggles remain with the codebase much longer than we expect. This is especially true for the A/B test toggles that last at least for the period of A/B test which might be long.

Using the feature toggles among business logic ties us closer to the library of choice. Changing it becomes risky.  
Think of a way to unit-test this code. The mechanism of enabling and disabling the toggles is usually completely hidden from us. Each unit test should then use some hacks to switch the toggles. Although these hacks are sometimes provided by the library, they tie us even harder to unnecessary implementation details.

If you're trying to build a reactive service, you might also be interested in other implementation details of the feature toggle library. For instance, a pretty common [togglz](https://www.togglz.org/) library allows getting the feature state from RBDMS storage. It provides a [JDBCStateRepository](https://github.com/togglz/togglz/blob/master/core/src/main/java/org/togglz/core/repository/jdbc/JDBCStateRepository.java) that uses blocking jdbc driver and makes a call to the database each time we want to know the feature state. It's not convenient enough, that's why the library provides [CachingStateRepository](https://github.com/togglz/togglz/blob/master/core/src/main/java/org/togglz/core/repository/cache/CachingStateRepository.java) that allows you to wrap JDBCStateRepository and make blocking calls only when the cache entry for the feature state expires or doesn't exist, which leads to random thread pool exhaustion at runtime.

Can we do better than this? Yes, for sure.

## Extracting Configuration from Business Logic

The first step is to define an interface for switchable components. In our case, we just need to extract the filtering function. If a switchable part contains multiple statements, we can encapsulate it in an object or a function that implements the interface.  

{% gist 8f95e2f41c0af1afefd9fce6eb530d79 %}

Next, we extract the switching logic to the place where the Cars object is created. In Spring-based apps, it's a factory method of a configuration class.

{% gist ccccd9ab6b10ad422e53151902e7898c %}

Since the domain code no longer depends on the configuration, this approach has multiple benefits:
- business logic can be easily tested separately from the configuration code and requires no mocking whatsoever;
- all configuration is in one place outside of the business logic, which gives us a better view on the way our application is configured and clear actionable feedback if the number of configurations grows too much;
- we can easily change the approach to dynamic configuration without a huge risk of breaking the domain code;

The ultimate goal of this technique is to refine the domain model and make it independent of the infrastructure. Practically, it means that dependency flow points inwards from infrastructure code to domain. 

![alt text](http://bit.ly/2DbxnJI?style=centered "diagram")

In this guide we explored how we can isolate the domain code from the dynamic cofiguration. But we can apply the same technique for other infrastructural aspects of our applications. Hope to cover them in the future articles, stay tuned!
