---
layout: post
date: 2018-10-26
title: "Demystifying the Magic of Spring: @Async"
description: |
keywords:
  - async
  - spring
  - refactoring
categories: refactoring
urlimage: 
published: true
---

Spring Framework is widely known for its magic. It’s a holy grail for junior Java developers, pride for some seniors and shame for the others. Spring combines old hacks that once made Java the privileged language of enterprise software development. However, most of these hacks are no longer needed and only lead to unmaintainable code.  
One of these hacks is the [@Async](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/annotation/Async.html) annotation that makes methods asynchronous under the hood.  
Let's discuss the problems with @Async and explore the modern alternative way to do asynchronous computations in Java.

<!--more-->

## Using @Async

Configuring @Async is easy. Add [@EnableAsync](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/annotation/EnableAsync.html) to your Spring configuration and enjoy its power.  

Let’s start with the simplest case - running a void method asynchronously.  

{% gist eda181b677c59606568f30e88d164ecb %}

As we expected, this method will run asynchronously. However, there is a nuance. @Async annotation has one parameter - the name of the executor that runs the annotated method. We haven't specified any executor, therefore the [SimpleAsyncTaskExecutor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/task/SimpleAsyncTaskExecutor.html) will be used by default. Guess what it does. **It fires up a new Thread for each method execution!** It might be a reasonable default for a hello-world app, but it's absolutely inappropriate for production.

Let's define an executor bean in our Spring configuration and use its name as the @Async annotation parameter.

{% gist d429c0ecba5d43f3b8c5ff41365c728b %}

Now we're in better control of resources allocated to async computations in our app.  

**What if somebody accidentally changes the name of the executor bean or makes a mistake in the @Async argument value?**  
We'll figure it out neither at compile time nor at the application startup. Instead, we'll get a [NoSuchBeanDefinitionException](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/NoSuchBeanDefinitionException.html) just when we call the async method for the first time. Hopefully, we have integration tests that will catch this problem. But in practice, the async effect is often disabled in favor of easier and more predictable testing, or Spring configuration for testing differs too much from the production one.

**What if we annotate private method with @Async?**  

By default, private methods will be executed synchronously without any warning due to the limitations of Spring AOP.
The only way to overcome this limitation is by turning on compile-time weaving.  

**What if the async method throws an uncaught exception?**  

Client object won't see this exception because it will be thrown in a different thread. But Spring cares about us providing an [AsyncUncaughtExceptionHandler](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/aop/interceptor/AsyncUncaughtExceptionHandler.html) hack to handle these unforunate exceptions.  

{% gist df4ac8cd52cba762b017144424b81c2b %}

I would argue that this error handling logic is too far away from the accident and is not as efficient as the error handling on the Client side could be.  

**What if somebody decides to return a raw value from the @Async method?**  

{% gist bc4e113ace1f9e24190d446bf7bd6698 %}

Interestingly, the method will run asynchronously immediately returning **null** to the caller. **NullPointerException** is just a matter of time.

We can solve it only by wrapping the return value in the [Future](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Future.html). However, this approach introduces the problem of dealing with a terrible (synchronous and non-composable) [Future.get(_)](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Future.html#get()) API on the client side.

{% gist c1037cb2730292f17c350a13053c48e4 %}

Unfortunately, this is the best we can get with @Async.  
Combining it with the poor testing support that requires Spring context initialization and configuration juggling, we get maintenance problems resulting in unpredictable behavior in production.
Hopefully, there is a solution.  

## Modern Approach to Async Computations in Java

@Async annotation was introduced in Spring 3.0. It was December 2009, Java 6 era with a completely broken Future abstraction.  
5 years later Java 8 released with a game-changing [CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html) API.  
CompletableFuture abstracts a composable asynchronous computation. Let's use it in our example.

{% gist 6b27b9a628ea0388526784c167a6ccb0 %}

You see, there is no magic.  
AsyncObject is a regular Java object with a mandatory explicit dependency on the executor defined in the constructor. Each method has predictable behavior, there are multiple ways to compose them, and exception handling is intuitive.  

But what about testing?  

{% gist a60561420b57bf68c925f39056a9e0f2 %}

Yes, it's a simple and fast unit test we all missed so much after using Spring for a long time.  

After all, it's no longer a question of whether to use @Async or CompletableFuture.  
Learn more about CompletableFuture and enjoy programming in Java.

**Resources:**
- I've built a GitHub project for you to experiment with both implementations: [https://github.com/IlyaZinkovich/spring-async-problems](https://github.com/IlyaZinkovich/spring-async-problems).
- [Guide to CompletableFuture](https://www.baeldung.com/java-completablefuture)
- [How to do @Async in Spring](https://www.baeldung.com/spring-async)
