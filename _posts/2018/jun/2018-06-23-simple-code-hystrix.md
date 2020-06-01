---
layout: post
date: 2018-06-23
title: "Simple Code: The Hystrix Story"
description: |
keywords:
  - simple made easy
  - hystrix
  - maintainability
categories: refactoring
urlimage: 
---

Writing maintainable code is definitely hard. It's even hard to define what "maintainable" means.  
After watching an enlightening [talk by the author of Clojure lang](https://www.infoq.com/presentations/Simple-Made-Easy/), I’d say that maintainable code should primarily be simple.  

> Simple is often erroneously mistaken for easy. "Easy" means "to be at hand", "to be approachable". "Simple" is the opposite of "complex" which means "being intertwined", "being tied together". Simple != easy.  
> -- Rich Hickey, <cite>[Simple Made Easy, 20 October 2011](https://www.infoq.com/presentations/Simple-Made-Easy/)</cite>. 

Reality is that we prefer easy "at hand" solutions that look innocent but lead to a poor maintenance experience.  
Let’s explore how simple code differs from easy in practice using Hystrix library as an example.  

<!--more-->

## Hystrix

[Hystrix](https://github.com/Netflix/hystrix/wiki) is a latency and fault tolerance library that became popular in Java community when it was open sourced by Netflix and integrated with [Spring Cloud](https://cloud.spring.io/spring-cloud-netflix/).  

One of the most popular use-cases for Hystrix is protecting your app against failures in communication with 3rd party services.

![alt text](https://bit.ly/2tnpQ6L?style=centered "use case")

Hystrix implements [Circuit Breaker](https://martinfowler.com/bliki/CircuitBreaker.html) pattern that helps you to isolate failure and deal with it proactively.

## Community-Driven Approach

Hystrix wouldn't succeed if it wasn't easy to use.  
In Java community "easy" means that you add an annotation and it works out of the box. And it really works:

{% gist b1df73ad08898b8d34d8008d10a0bb27 %}

`callWebService()` method will return a default response if a web service throws an exception and you'll see the errors statistic on a picturesque dashboard. Additionally, you can configure Hystrix command in a way that it won't even call the web service after a specified amount of errors.   
Looks awesome and, without a doubt, it's easy.  

Let's test it. How can we do this?  
You read [hystrix-javanica](https://github.com/Netflix/Hystrix/blob/master/hystrix-contrib/hystrix-javanica/README.md) documentation and realize that this code works only under certain conditions. You need to understand [aspect-oriented programming model](https://en.wikipedia.org/wiki/Aspect-oriented_programming), [AspectJ](https://www.eclipse.org/aspectj/) and [Spring AOP](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aop) (if you're using Spring) to make it work.  

The solution is to define a Spring context with AspectJ auto proxy and create App object as a bean.

{% gist e3571a2267550473bacf423e5a14c601 %}

The test looks as follows.  

{% gist 4f0c43b7f836703a9cf22f202efa60b8 %}

Although we easily implemented basic fallback functionality, we added accidental complexity that leads to maintenance problems:
- Compiler won't help you. Values that you specify in `@HystrixCommand` annotation parameters are plain strings. Imagine what will happen if somebody renames the fallback method. You need tests to prevent these accidents.  
- It's hard to debug this code. The entry point into Hystrix functionality is hidden in the aspect that is not a part of your source code. An inexperienced developer can spend hours or even days trying to find it. Moreover, it's not uncommon that the core library works fine, but the aspect that ties it to your code has a bug. This fact doesn't allow you to concentrate only on core functionality during debug session.  
- The code is fragile. There are too many pieces that need to be carefully arranged to make this code work. And it's so easy to miss something in a constantly changing system.  

## Simple Approach

First thought is that Hystrix is just a poorly designed library. But surprisingly it's the opposite.
After reading the [wiki](https://github.com/Netflix/Hystrix/wiki/How-To-Use) it appears that there is a `HystrixCommand` class that has a well-defined interface.

Let's create a Hystrix command for our use case.

{% gist 104b8fe1f0b2f50cbb266aca8e1ecb58 %}

It's a generic implementation that can be reused for any command with a fallback. You write this once and reuse across the whole project.

The app code looks as follows.

{% gist 6164cf96fcfcf5496d4726fd71f24e5f %}

You can reduce command object creation to just one line.  
{% gist 9ff4976d2a4ab6615ca93a9875bc9135 %}

Sometimes it even makes sense to use a builder pattern with a rich DSL. You can get inspiration from another simple fault tolerance library - [Failsafe](https://github.com/jhalterman/failsafe).  

{% gist c9296b575ef2f036bce74c2572da59d2 %}

Testing is simple. You don't need a Spring context and Spring testing components such as mock beans. All you need is just a plain old JUnit test.  

{% gist 1c8cec32a869382dc9c6675ba833e9b2 %}

Let's summarize what we just achieved:  

- Compiler is helping you. Any refactoring you can imagine will no longer break your app. Moreover, the fallback function is no longer marked as unused code. This reduces some compiler warnings and makes static code analysis more precise.  
- Debugging is straightforward. The entry point into Hystrix functionality is right behind your eyes, and it's just a simple Java object that is much more predictable than the aspect. If anything fails, you can quickly locate the failure.  
- The code is robust. We reduced the number of moving parts to just the core Hystrix library. Moreover, you can integrate this code into any Java application without any additional configuration. It's just hard to make a mistake.  

These all are the benefits the **simple** code gives us. 

Although big established frameworks like Spring strongly influence our perception of code quality, at the end of the day, it's our responsibility to decide which code to write. My decision is to write **simple** code.

Here is the link to the repo containing source code: 
[https://github.com/IlyaZinkovich/simple-hystrix](https://github.com/IlyaZinkovich/simple-hystrix)
