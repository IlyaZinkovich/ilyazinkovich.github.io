---
layout: post
date: 2018-11-03
title: "Demystifying the Magic of Spring: @Cacheable"
description: |
keywords:
  - cacheable
  - spring
  - refactoring
categories: refactoring
urlimage: 
published: false
---

Continuing the topic of [Demystifying the Magic of Spring](http://bit.ly/2OBlghz) I'd like to discuss the [@Cacheable](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/annotation/Cacheable.html) annotation. @Cacheable has all the problems of annotation processing in Java and Spring AOP as well as incompatibility with the modern reactive approach to building web applications.  
Anyway, it’s all for the best. And it’s great to see how the growing needs of our industry push us in better tools direction.  
If you're migrating to reactive web frameworks such as [Spring WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) or just getting tired of maintenance problems with @Cacheable this article is for you.

<!--more-->
