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

@Cacheable is another little thing we got used to while building Spring applications. 
If we want to cache a result of the method invocation we just annotate this method with @Cacheable and Spring will do the rest for us.

This approach sounds like a good idea up until you start working on a huge codebase.
You need to remember a couple of tricks to be confident that you don't break the app:
1. Every class you annotate with @Cacheable must be declared as a bean.  
2. Caching works only for public methods by default (have a good time with compile-time weaving for non-public methods).  
3. All parameters that you specify in the @Cacheable annotation will only be checked at runtime when the target method is called. If you accidentally changed the cache name you will get IllegalArgumentException when you call the method. You can use ugly Spring Expression Language (SpEL) to specify keys for cache entries, conditions for caching based on method parameters and return value. Compiler doesn't have a clue about SpEL. That's why it's so simple to make a mistake in one of these expressions.
4. Testing requires bootstrapping of Spring application context and Spring-specific test tools (autowiring beans into the test object, @MockBean, @SpyBean).
5. @Cacheable doesn't work for CompletableFuture or Mono/Flux.

Don't forget basic things such as making beans for all the classes that require Spring caching.
And keep in mind that Spring caching doesn't work by default for non-public methods.

@Cacheable is unpredictable. 
What does it mean to cache CompletableFuture or Mono/Flux?
What if we decide to use @Cacheable for generic class?

Doesn't work: [StackOverfow Question](https://stackoverflow.com/questions/36977643/spring-cache-not-working-for-abstract-classes)
Simply put [@Cacheable cannot cache Mono and Flux types](https://stackoverflow.com/questions/48156424/spring-webflux-and-cacheable-proper-way-of-caching-result-of-mono-flux-type)
If you're using caching technology like Redis or Memcached, @Cacheable will block your main thread.

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
