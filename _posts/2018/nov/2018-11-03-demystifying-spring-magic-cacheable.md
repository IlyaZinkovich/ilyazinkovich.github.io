---
layout: post
date: 2018-11-03
title: "Demystifying the Magic of Spring: Caching"
description: |
keywords:
  - cacheable
  - spring
  - refactoring
categories: refactoring
urlimage: 
published: false
---

Continuing the topic of [Demystifying the Magic of Spring](http://bit.ly/2OBlghz), this time I'd like to discuss [Caching](https://www.baeldung.com/spring-cache-tutorial). Caching in Spring has all the problems of annotation processing in Java and Spring AOP as well as incompatibility with the modern reactive approach to building web applications.  
Anyway, it’s all for the best. It’s great to see how the growing needs of our industry push us in better tools direction.  
If you're migrating to a reactive web framework such as [Spring WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) or just getting tired of maintenance problems with @Cacheable and related annotations, this article is for you.

<!--more-->

## Maintenance First

Configuring caching in Spring is a rather easy task.

What if we decide to use @Cacheable for generic method?

Doesn't work: [StackOverfow Question](https://stackoverflow.com/questions/36977643/spring-cache-not-working-for-abstract-classes)

## Caching in Reactive Web

Simply put [@Cacheable cannot cache Mono and Flux types](https://stackoverflow.com/questions/48156424/spring-webflux-and-cacheable-proper-way-of-caching-result-of-mono-flux-type)
