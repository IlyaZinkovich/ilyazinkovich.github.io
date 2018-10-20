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
published: false
---

Spring Framework is widely known for its magic. It’s a holy grail for junior Java developers, pride for some seniors and shame for the others. Spring combines old hacks that once made Java the privileged language of enterprise software development. However, most of these hacks are no longer needed and only make things worse.  
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

**What if the async method throws an uncaught exception?**  

Client object won't see this exception because it will be thrown in a different thread. Spring cares about us providing an [AsyncUncaughtExceptionHandler](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/aop/interceptor/AsyncUncaughtExceptionHandler.html) hack to handle these unforunate exceptions.  

{% gist df4ac8cd52cba762b017144424b81c2b %}

I would argue that this error handling logic is too far away from the accident and is not as efficient as the error handling on the Client side could be.  

**What if somebody decides to return a raw value from the @Async method?**  

{% gist bc4e113ace1f9e24190d446bf7bd6698 %}

Interestingly, the method will run asynchronously immediately returning **null** to the caller. **NullPointerException** is just a matter of time.

We can solve it only by wrapping the return value in the [Future](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Future.html). However, this approach introduces the problem of dealing with a terrible (synchronous and non-composable) [Future.get(_)](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Future.html#get()) API on the client side.

{% gist c1037cb2730292f17c350a13053c48e4 %}

Unfortunately, Spring doesn't provide any alternative.  

## Here is a quick summary why @Async is dangerous:
1. Unsafe to use with a default executor.
2. Late configuration problem detection.
3. Poor exception handling.
4. Doesn't work with private methods.
5. Testing requires initializing Spring context and Spring-specific testing tools.
