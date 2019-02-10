---
layout: post
date: 2019-02-06
title: Isolating Business Logic from Dynamic Configuration
description: |
keywords:
  - modular design
  - configuration
categories: modular design
urlimage: 
published: true
---

Modern businesses demand higher flexibility of our applications.  
Emerging data-driven approaches to product development encourage A/B testing to improve product-market fit gradually.  
Continuous Delivery popularises decoupling of feature releases from the deployment cycle.  
All these techniques require some sort of dynamic configuration. Poorly implemented, the dynamic configuration makes the code fragile. Being not able to change the software frequently we lose all the benefits of delivering continuously and being data-driven.  
However, we can mitigate these issues if we stop mixing configuration code with business logic.  

<!--more-->

## Common Approach to Dynamic Configuration

[Feature Toggles](https://martinfowler.com/articles/feature-toggles.html) are widely used to configure the applications dynamically. Here is an example of **how** they're used.

Let's say we're developing a car-sharing service and need to get cars nearby client location filtered according to his preferences. We have a couple of filters:
- **prodFilter** that filters cars according to the last stable version of a filter;  
- **candidateFilter** that was developed today and should be switched on and off with a feature toggle and finally replace the prodFilter;  
- **awesomeFilter** which is an experimental filter that targets a specific client group according to A/B test configuration based on the client id.  

Additionally, somebody who integrated Prometheus into our service added a counter that's incremented if no cars are presented to the clients and hid this change behind a toggle as well.    

{% gist e29341606c2106bb4871bc6bdf938794 %}

We're used to this kind of code style. It's easy to write until you cannot recognize what's the business logic of the code hidden behind these nested if-else statements that appear throughout the whole codebase.
Ideally, this situation should never happen because we all agree that the toggles are temporary and should be removed as soon as the switching is finished. But as practice shows, stale toggles remain with the codebase much longer than we expect. This is especially true for the A/B test toggles that last at least for the period of A/B test which might be long.  

The domain class knows about the ongoing A/B testing process, the rollout of the new functionality and half-ready Prometheus integration. Should anyone who tries to understand and change "the logic of presenting the personalized set of cars" know about all this stuff?  

Look a the last if statement.  

{% gist 4ead32f96cef2faf826474133c078134 %}

The mix of the toggle and business conditions blurs the boundaries between the infrastructure and domain code. After Prometheus integration is complete, you need to be careful about removing all the toggles in a way that the business logic remains the same. The risk of messing it up grows with the amount and complexity of the code coupled to the toggle.  

Think of a way to unit-test this code. The mechanism of enabling and disabling the toggles is usually completely hidden from us. Each unit test should then use some hacks to switch the toggles. Although these hacks are sometimes provided by the library, they tie us even harder to unnecessary implementation details. Anyway, each test method will contain the magic combination of enabling and disabling of all the toggles that are used in a target class.  

{% gist 41e3bb5b6a3a5d44e1165c597c73609d %}

Can we do better than this?  

## Extracting Configuration from Business Logic

The first step is to define an interface for switchable components. In our case, we just need to extract the filtering function. If a switchable part contains multiple statements, we can encapsulate it in an object or a function that implements the interface.  

{% gist 8f95e2f41c0af1afefd9fce6eb530d79 %}

Next, we extract the switching logic to the place where the Cars object is created. In Spring-based apps, it's a factory method of a configuration class.

{% gist ccccd9ab6b10ad422e53151902e7898c %}

Since the domain code no longer depends on the configuration, this approach has multiple benefits:
- business logic becomes crystal clear and free from infrastructure concerns;
- we can test the domain code separately from the configuration code without any mocking and toggle enabling/disabling boilerplate;  
- all configuration is now in one place (outside of the business logic) which gives us a better view on the way our application is configured, easy toggle removing procedure and actionable feedback if the number of configurations grows too much.  

The ultimate goal of this technique is to clean up the domain modelling code and make it independent of the infrastructure. Practically, it means that dependency flow points inwards from infrastructure code to domain. 

![alt text](http://bit.ly/2DbxnJI?style=centered "diagram")

In this guide, we explored how we can isolate the domain code from the dynamic configuration. In fact, we can apply the same technique for other infrastructural aspects of our applications. Hope to cover them in the future articles, stay tuned!
