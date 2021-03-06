---
layout: post
date: 2019-05-01
title: "How To Make Sustainable Shared Libraries"
description: |
keywords:
  - shared libraries
  - design
categories: design
urlimage: 
published: false
---

When the amount of code grows to a certain point or some cross-cutting concern should necessarily be applied to a wide range of services, developers start thinking about the ways to share the code. Although a lot of common infrastructure concerns are gradually extracted into side-car containers, shared libraries are still commonly used to share valuable assets across the whole company. They are easy to start with and can quickly go bad turning into a silent time-bomb for that exact wide range of services that integrated with it.  
Let’s discuss how we can develop better libraries that give clients full control over what their services do and do not.  

<!--more-->

## The Common Approach

Shared libraries developed in-house tend to be extremely easy to integrate with. What I usually hear when somebody pitches his creation is along the lines of “just add this dependency to your project” and sometimes also “just add this annotation to your class” and everything just works. It’s an amazing experience, isn’t it?  

If you ever tried to debug these libraries when they unexpectedly stop to work or even wrote them, you know that they’re mostly relying on some cryptic extension mechanism of a framework that your company generally uses for application development. In Java, this framework usually happens to be Spring (Boot).  

Spring Boot’s main (and pretty much single) feature is auto-configuration. For each Spring library (Web, Data, Test and many more) there is a corresponding Spring Boot Starter that auto-configures the library components based on application properties. As a result, you can easily bootstrap your Java web app with just a few lines of code.  

And then a natural idea comes to our mind - let’s use the same extension mechanism for our in-house libraries.  

Here’s what we get:
1. The library starts to depend on a specific Spring and Spring Boot version which complicates the framework upgrade on the client side. Although the high-quality migration guide for Spring libraries is usually available, we mostly never have this luxury for the in-house libraries. Even Spring Team itself has this problem - Spring Cloud libraries have a completely different versioning scheme than Spring or Spring Boot, and each library has an independent release cycle. Narrowing all the Spring-based libraries down to the common version of a framework becomes tricky, boring, time-consuming and we hardly ever have full confidence that everything will work at the end.  
2. The library’s configuration changes over time. New properties are added, old ones are accidentally renamed or removed. Ideally, you’d expect your application to fail on startup if you don’t supply the value for the required properties, but most of the time the library within your app will continue to operate with some inappropriate defaults. (this is the common disappointing experience you get while using Spring Boot).  
3. The library gets access to all the application properties and therefore can form invisible dependencies on any property that your application has or even override them without you even knowing about it. Sometimes it's even expected if you're using Spring library to integrate with a remote property source like Vault, AWS Secrets Manager and others, and then you can hardly reason about how your application is configured.  
4. Most importantly, with this approach to library development you hardly ever know which components your application has and what each of them is doing. Discovering these components from your codebase becomes very difficult because sometimes you don’t even use the classes from the library directly and cannot easily find and debug them (remember some unexpected Servlet Filters or HTTP Iterceptors silently added to your app).  

Let's try out the alternative approach that will help you to avoid these problems and enjoy reusing the code.  
As an example we'll build a library that provides [an alternative to Spring Caching](http://bit.ly/2UmsOYz) for asynchronous code.  

## Define the Value of Your Library

Seriously, formulate in one sentence why somebody should consider using your library and write it in README file.  

{% gist ce5390816e9b71df126d8779b04de112 %}

Congratulations! Our library got its first valuable piece of documentation.  

## Codify the Core

Define the core abstractions and implement them using the standard library of your programming language. Avoid adding any external dependencies at this point.  

You might think that it's imposible to build anything valuable this way, but let's do this exercise.  

First, we'll define the core caching abstraction.  

{% gist 32d571d480423c896e1087630f6baef2 %}

And then we'll implement a version of it that caches the serialized result of asynchronous computation in a remote store.  
