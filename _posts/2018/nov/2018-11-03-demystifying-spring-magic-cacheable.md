---
layout: post
date: 2018-11-03
title: "Taking Control over Caching in a Spring App"
description: |
keywords:
  - cacheable
  - spring
  - refactoring
categories: refactoring
urlimage: 
published: false
---

Continuing the topic of [Magic in Spring Framework](http://bit.ly/2OBlghz), I'd like to discuss [Caching](https://www.baeldung.com/spring-cache-tutorial). Caching in Spring has all the problems of annotation processing in Java and Spring AOP as well as incompatibility with the modern reactive approach to building web applications.  
Anyway, it’s all for the best. It’s great to see how the growing needs of our industry push us in better tools direction.  
If you're migrating to a reactive web framework such as [Spring WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) or just getting tired of maintenance problems with @Cacheable, this article is for you.

<!--more-->

## What's wrong with @Cacheable?

@Cacheable is another little thing we got used to while building Spring-based apps. If we want to cache a result of the method invocation we just annotate this method with @Cacheable and Spring will do the rest for us.  
This approach sounds like a good idea until you start working on a huge codebase.  
The main problem is the same as with @Async - it's just too easy to make a mistake leaving it unnoticable until it's too late.  
You need to remember a couple of tricks to be more or less confident when you use @Cacheable:  
1. Every class you annotate with @Cacheable must be declared as a bean.  
2. Caching works only for public methods by default (compile-time weaving required for non-public methods).  
3. All parameters that you specify in the @Cacheable annotation will only be checked at runtime when the target method is called. If you accidentally changed the cache name you will get IllegalArgumentException when you call the method. You can use ugly Spring Expression Language (SpEL) to specify keys for cache entries and conditions for caching based on method parameters and return value. Compiler doesn't have a clue about SpEL. Each mistake in a SpEL expression will result in a runtime exception.  
4. Testing requires the bootstrapping of Spring application context and Spring-specific test tools (autowiring beans into the test object, @MockBean, @SpyBean).  
5. @Cacheable doesn't work for Mono and Flux.  

## Pure Java Alternative to @Cacheable

Spring is widely known for its "declarativeness". @Cacheable annotation declares **what** should be cached instead of **how**. It decouples the business logic of your application from concrete caching implementation. Sounds good.  
However, Java always had a good tool for declarative programming - interfaces. Essentially, what @Cacheable annotation does can be declared in a simple interface.

{% gist a2849dd18847a5c85481e487fcf00b95 %}

For example, we have a class that takes a string, calls a remote service and returns a response to the client. 
Caching the remote service response with an input string as a cache key will look as follows:

{% gist fee149f488b4a133d9d063cf3cc76d4d %}

What if our remote service returns Mono&lt;String&gt; and we want to cache it in Redis?

Let's cover a more complicated case - cache results of a method returning Mono in Redis. 
Mono encapsulates lazy asyncronous computation of 0..1 results. If you subscribe to the same Mono twice, then the computation will run twice too. Therefore, there is no sense to cache Mono instances with @Cacheable. Instead, we'd like to cache the results of the computations encapsulated in Mono. Using interfaces we can define these caching semantics ourselves. 

Full implementation using [Lettuce](https://lettuce.io) reactive client for Redis will look as follows:

{% gist 14f5eae0c2fbcab7501e6dd916fde7d9 %}
