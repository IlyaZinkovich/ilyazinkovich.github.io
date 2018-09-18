---
layout: post
date: 2018-09-21
title: "Asynchronous Communication with Legacy System"
description: |
keywords:
  - async
  - legacy
  - distributed systems
categories: distributed systems
urlimage: 
published: false
---

Implementing anything inside a legacy codebase can be painful. 

>"Legacy code. The phrase strikes disgust in the hearts of programmers. Although our first joy of programming may have been intense, the misery of dealing with legacy code is often sufficient to extinguish that flame."  
>[Michael Feathers, Working Effectively with Legacy Code](https://www.amazon.de/Working-Effectively-Legacy-Robert-Martin/dp/0131177052).  

That's why we, developers, naturally tend to implement new functionality outside of it. With the rise of microservices, this tendency escalated to a whole new level. Anytime we want to implement something new, we think of creating a new web service. Applying this approach regularly, we add more moving parts. Losing control over this process, we risk creating a highly unreliable system.  

We've already discussed how to sustain [Reliable Services Communication]({% post_url 2018/jun/2018-06-02-reliable-services-communication %}) for relatively new services.
But what if we're stuck with a legacy system that everyone is scared to change for some valid reasons?

<!--more-->

## Asynchronous Communication with Legacy System
