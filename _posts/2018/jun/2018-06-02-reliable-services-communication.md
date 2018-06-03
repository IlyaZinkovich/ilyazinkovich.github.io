---
layout: post
date: 2018-06-02
title: "Designing Reliable Services Communication"
description: |
keywords:
  - http
  - event stream
  - distributed systems
categories: distributed systems
---

Services communication is a fundamental part of any distributed system design. There are multiple ways you can make services communicate, and all of them are either synchronous or asynchronous. The choice of which one to use is essential. It's not just about choosing sync HTTP client for the ease of development and then switching to async one to make service perform better. This choice defines system's reliability. Let's design the same system using both approaches to services communication and compare them.

<!--more-->

## Incentives and Payments example

Suppose we're building a delivery service in which we pay couriers per delivery. Payments service is already in place and is busy transferring money to couriers' accounts. To improve quality of service, we decided to encourage couriers giving them monetary bonuses and increasing payments based on their performance. Just for this purpose, we start building Incentives service.

## Let's design it the synchronous way

Incentives service needs to expose API for giving couriers various types of bonuses and keeping track of their performance.
While sketching a simple system level diagram, we decide that Incentives service will synchronously call Payments when we need to pay money for a bonus and Payments will call Incentives when it needs to know couriers' performance to make payment amount adjustment.  

![alt text](https://bit.ly/2HfpH9r?style=centered "synchronous communication design")

The services code looks pretty straightforward and very similar to in-process communication.

{% gist f6c30ead18006c269771e0e16e6487c0 %}

{% gist 050fffd3085f0ee9d354d34adf93984d %}

Everything works fine until Payments service goes down and we start preparing the workaround to batch unpaid bonuses on the client side until Payments goes live.  
After we fix this unfortunate issue, Incentives service starts failing. How much do we need to pay couriers if we cannot obtain their performance? Do we need to delay the payments until Incentives service is back?    
These inherent problems remain unnoticeable until they result in system outages or drastically slow down the development cycle.

## Asynchronous way

Let's ask ourselves the following questions:
- Is Incentives service responsible for making sure the money is paid for the bonus?
- If Payments service is so interested in couriers performance, shouldn't it own this data instead of asking for it each time?

This leads us to a different kind of architecture where Incentives service notifies the rest of the system of events happening in this service.

![alt text](https://bit.ly/2LiEt1z?style=centered "asynchronous communication design")

If Payments service goes down, event stream will naturally buffer the bonuses.   
If Incentives service goes down, Payments will continue working using couriers performance information obtained from the recent events and cached locally. Surprisingly, we even achieved better performance removing unnecessary network call.  
We modeled the corner cases as part of the natural flow without adding the messy workaround code.  
Moreover, we redistributed responsibilities between these services in a slightly different way. Incentives service no longer has any payment-related code and Payments service owns all the information needed to make payments.  
Asynchronous communication enabled true decoupling of services in space and time contributing to better system reliability while keeping the code clean.

{% gist 3188166e5a636c840868ff072fa7103f %}

{% gist 11691a92cecddc740ca6fb122c1d42e7 %}

## Summary

Synchronous services communication is easy to understand. It closely resembles the function calls made inside a process on a single machine. But this simple concept breaks when we try to apply it to distributed systems design.  
Although a lot of the systems are designed in this way, we can start using asynchronous services communication making them inherently more reliable.  

Here is a link to the repository containing source code: https://github.com/IlyaZinkovich/event-sourcing-consistency
