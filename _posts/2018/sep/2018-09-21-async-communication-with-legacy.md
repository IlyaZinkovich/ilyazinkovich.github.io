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

Implementing anything inside a legacy system can be painful. 
That's why we, developers, naturally tend to implement new functionality outside of the old codebase. 
With the rise of microservices, this tendency escalated to a whole new level. Anytime we want to implement something new, we think of creating a new web service. Applying this approach regularly, we add more moving parts. Losing control over this process, we risk creating a highly unreliable system.
We've already discussed how to sustain [Reliable Services Communication]({% post_url 2018/jun/2018-06-02-reliable-services-communication %}) for relatively new services.
But what if we're stuck with a legacy system that we're not allowed to change too much?

<!--more-->

## Asynchronous Communication with Legacy System
