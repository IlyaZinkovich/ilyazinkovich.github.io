---
layout: post
date: 2019-03-03
title: "Caching in Spring is Inherently Broken"
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
If you're migrating to a reactive web framework such as [Spring Web`Flux`](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) or just getting tired of maintenance problems with `@Cacheable`, this article is for you.
`@Cacheable` is a little pleasure we got used to while building Spring-based apps. If we want to cache a result of the method invocation we just annotate this method with `@Cacheable` and Spring does the rest for us.  

<!--more-->

## Caching with Spring by Example

We'll challenge this statement while implementing a very primitive wrapper for the async HTTP client that caches responses in Redis.  

{% gist fdfb5d528e65dad2ba93884e00d9146f %}

Let's test the caching mechanism with a simple `ConcurrentMap` in-memory cache to verify that responses are actually being cached per a combination of a request method and url.  

For the purpose of testing we'll use the `HttpClient` implementation that returns a Response containing a randomly generated string.  

{% gist 318617d61c0b1fe4cbb21f985ec663fe %}

Here is the config.  

{% gist 85a9853d5540e3fe64affc3c8c3db8b0 %}

And here is the test. 

{% gist 61a18837cadc91a1d84e4b9a371b1648 %}

Test results are successful as expected. We already got some confidence, let's just replace `ConcurrentMap` with Redis and ship the code to production.  

The only thing that needs to change, [as per Spring documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-caching.html), is the configuration.  
With latest Spring Boot it should work like magic, but let's look under the hood and configure the `RedisCacheManager` ourselves.  

{% gist 9fec9b569a628e9566c9dc280caf1399 %}

In contrast with a `ConcurrentMap`, Redis is a remote service and we need to configure serialization to be able to exchange data with it. Keys of the cache entries will be simple String's and values will be serialized as JSON.  
The only working configuration of the JSON serializer stores a class name of a serialized object in a payload. If we forget (or refuse) to configure it this way, we'll get the following error.  

{% gist a345aa16a3aefa5c2f6e65a6d5f08456 %}

This approach has one important drawback. If somebody moves or renames the class of the objects cached in Redis we're screwed - deserialization will fail and the method call will result in runtime exception (**the method won't be called without caching, it will simply blow up**).  

If you wonder what is the default that Spring Boot uses for serialization, you'd be very surprised that it's the built-in [Java serialization](https://docs.spring.io/spring-data/redis/docs/current/api/org/springframework/data/redis/serializer/JdkSerializationRedisSerializer.html) which is slow, error-prone, verbose, insecure and has backward compatibility issues.  

Hmm... Our JSON serialization is at least better that the default. Forget everything, let's run the test with the switched config. This is what we get:  

{% gist e3e3b833d2ef487ec1413e239aa74e2b %}

## What's Wrong?

Although `CompletableFuture` is a backbone of async programming in Java, Spring doesn't handle it in a special way. Spring tries to serialize it and fails. This issue is relevant to caching in any external datastore. [The corresponding issue](https://github.com/spring-projects/spring-framework/issues/17559) was reported to the Spring Framework team and closed without a fix.   

But why does the in-memory version work???  
Since serialization isn't required, the instance of `CompletableFuture` is successfully saved in a `ConcurrentMap` as any other Java object. Now imagine what happens if the asynchronous computation behind the cached `CompletableFuture` fails. The failed `CompletableFuture` is returned from cache until the TTL expires. In practice, the impact of this problem ranges from the intermittent degradation of the application functionality up to severe incidents depending on how close the caching is to the critical path.  

If we replace `CompletableFuture` with a new shiny reactive `Mono` or `Flux` types the situation becomes different. Since `Mono` and `Flux` are just describing the asynchronous computation, the in-memory cache will store the description (and not the result) and the subscribers will re-run the computation each time they receive its description from cache - as if there was no cache at all. Useless but safe. However, with a remote cache the application will still crash.  

If you're still being optimistic about Spring caching, keep in mind the following:  
1. Spring abstractions for caching are very primitive and are not suitable for even slightly more complicated use cases such as caching of multiple values inside a single method, caching of async results, dynamic TTLs, custom caching strategies like parallel cache lookup and caching in remote datastores is by-design prone to backward compatibility errors. Moreover, even the simplest functionality that Spring provides is hard to debug - a bunch of functionality is hidden behind autogenerated proxies in legacy trenches of 15 years old framework.  
2. Spring AOP. Every class you annotated with `@Cacheable` must be declared as a bean and caching will work only for public methods called from the outside of the class (compile-time weaving is required for non-public methods).  
3. [Spring Expression Language (SpEL)](https://www.baeldung.com/spring-expression-language). `@Cacheable` annotation parameters can optionally contain SpEL expressions. SpEL can be used to define dynamic keys and caching conditions based on method parameters. Java compiler doesn't have a clue about SpEL. Each mistake in a SpEL expression will result in a runtime exception.  
4. Spring Test. Testing requires the bootstrapping of Spring application context and Spring-specific test tools that make your test suite unintentionally complicated and slow.  

## What Can We Do Better?

We can replace `@Cacheable` with an interface representing required caching capabilities and inject it into our client class.  

{% gist f8af40db43bb1855224afa50f3eace40 %}

The cache key is now defined in a type-safe way. The silliest mistakes will be catched by the compiler.  
The expression producing a cached value is now turned into a `Supplier<CompletableFuture<T>>`. It can be evaluated on-demand by `Cacheable` implementation if the cached value is not found.  

I'd rather avoid Spring caching altogether.  
We can define concrete caching capabilities that we need using Java interfaces.
Here is an example of the `ConcurrentMap`-based implementation for caching `CompletableFuture` results in our `CachingHttpClient`.

{% gist 4d0898358e058f4623cd94e58367f6c9 %}

There is nothing innovative in this approach. I'd just argue that this is not as complicated and one might see it.  
