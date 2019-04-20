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

When the amount of code grows to a certain point or some cross-cutting concern should necessarily be applied to a wide range of services, developers start thinking about the ways to share the code. Although a lot of common infrastructure concerns are gradually extracted into side-car containers, shared libraries are still commonly used to share valuable assets across the whole companies. They are easy to start with and can quickly go bad turning into a silent time-bomb for that exact wide range of services that integrated with it.  
Let’s discuss how we can develop better libraries that give clients full control over what their services do and do not.  

<!--more-->

## What we are trying to fix

Shared libraries developed in-house tend to be extremely easy to integrate with. What I usually hear when somebody pitches his creation is along the lines of “just add this dependency to your project” and sometimes also “just add this annotation to your class” and everything just works. It’s an amazing experience, isn’t it?  

If you ever tried to debug these libraries when they unexpectedly stop to work or even wrote them, you know that they’re mostly relying on some cryptic extension mechanism of a framework that your company generally uses for application development. In Java, this framework usually happens to be Spring (Boot).  

Spring Boot’s famous and pretty much single feature is auto-configuration. For each Spring library (Web, Data, Test and many more) there is a corresponding Spring Boot Starter that auto-configures the library components based on application properties. As a result, you can easily bootstrap your Java web app with just a few lines of code.  

And then a natural idea comes to our mind - let’s use the same extension mechanism for our in-house libraries. 
Here’s what we get:
1. The library starts to depend on a specific Spring and Spring Boot version which complicates the framework upgrade on the client side. Although the high-quality migration guide for Spring libraries is usually available, we usually don’t have this luxury for the in-house libraries, especially for the most abandoned ones. Even Spring Team itself has this problem - Spring Cloud libraries have a completely different versioning scheme than Spring or Spring Boot, and each library has an independent release cycle. Narrowing all the Spring-based libraries down to the common version of a framework becomes tricky, boring, time-consuming and we hardly ever have full confidence that everything will work at the end. 

