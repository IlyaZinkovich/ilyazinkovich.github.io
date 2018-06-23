---
layout: post
date: 2018-06-22
title: "Simple Made Easy: The Hystrix Story"
description: |
keywords:
  - simple made easy
  - hystrix
  - maintainability
categories: refactoring
urlimage: 
---

Writing maintainable code is hard. But what makes code maintainable?  
After watching an enlightening [talk by the author of Clojure lang](https://www.infoq.com/presentations/Simple-Made-Easy/), I’d say that maintainable code should primarily be simple. But “simple” in a specific way.

>"Simple is often erroneously mistaken for easy. "Easy" means "to be at hand", "to be approachable". "Simple" is the opposite of "complex" which means "being intertwined", "being tied together". Simple != easy."  
>[Rich Hickey, 20 October 2011](https://www.infoq.com/presentations/Simple-Made-Easy/).  

Reality is that we prefer easy "at hand" solutions that look innocent but lead to a poor maintenance experience.  
Let’s explore what differs simple code from easy in practice using Hystrix library as an example.

<!--more-->

## Hystrix

[Hystrix](https://github.com/Netflix/hystrix/wiki) is a latency and fault tolerance library that became popular when it was open sourced by Netflix and integrated with Spring Cloud.  

The most popular use-case for Hystrix is protecting your app against the faults in 3rd party services.

![alt text](https://bit.ly/2tnpQ6L?style=centered "use case")

Hystrix implements [Circuit Breaker](https://martinfowler.com/bliki/CircuitBreaker.html) pattern that helps you to isolate failure and deal with it proactively.

## Use Case

Hystrix wouldn't have such success if it wasn't easy to use.  
In Java-world "easy" means you add an annotation and it works out of the box. And it really works:

{% gist b1df73ad08898b8d34d8008d10a0bb27 %}

`callWebService()` method will return a default response if a web service throws an exception and you'll see the errors statistic on a picturesque dashboard. Additionally, you can configure Hystrix command in a way that it won't even call the web service after a specified amount of errors. Looks awesome.  

No doubt it was easy to write this code.
Is this code simple?

"""
Java language has a great advantages over other languages such as reflection and annotations. All modern frameworks such as Spring, Hibernate, myBatis and etc. seek to use this advantages to the maximum. 
"""

And this is what makes these frameworks very complicated and error-prone.

This seems to be a good idea until you start maintaining this code.

Here is the link to the repo containing source code: 
[https://github.com/IlyaZinkovich/simple-hystrix](https://github.com/IlyaZinkovich/simple-hystrix)
