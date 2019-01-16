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
If you're migrating to a reactive web framework such as [Spring WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) or just getting tired of maintenance problems with @Cacheable, this article is for you.

<!--more-->

## What's wrong with @Cacheable?

@Cacheable is another little thing we got used to while building Spring-based apps. If we want to cache a result of the method invocation we just annotate this method with @Cacheable and Spring will do the rest for us.  
Here is an example of caching the expensive transformation result in a ConcurrentHashMap:

{% gist fdfb5d528e65dad2ba93884e00d9146f %}

Even this simple code works only under certain conditions.
The main problem is the same as with @Async - it's just too easy to make a mistake and leave it unnoticable until it's too late.  
You need to remember a couple of tricks to be more or less confident when you use @Cacheable:  
1. Every class you annotate with @Cacheable must be declared as a bean.  
2. Caching works only for public methods by default (compile-time weaving required for non-public methods).  
3. All parameters that you specify in the @Cacheable annotation will only be checked at runtime when the target method is called. If you accidentally changed the cache name you will get IllegalArgumentException only when you call the method. You can use ugly [Spring Expression Language (SpEL)](https://www.baeldung.com/spring-expression-language) to specify keys for cache entries and conditions for caching based on method parameters and return value. Compiler doesn't have a clue about SpEL. Each mistake in a SpEL expression will result in a runtime exception.  
4. [@Cacheable doesn't work for Mono and Flux](https://stackoverflow.com/questions/52871436/alternatives-to-cacheable-with-spring-webflux/52948705).  
5. Testing requires the bootstrapping of Spring application context and Spring-specific test tools (autowiring beans into the test object, @MockBean, @SpyBean).  

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
