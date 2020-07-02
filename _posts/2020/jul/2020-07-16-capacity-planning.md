---
layout: post
date: 2020-06-30
title: "Data-Driven Capacity Planning"
description: |
keywords:
  - deliveries
  - optimization
  - algorithms
categories: algorithms
urlimage: 
published: false
---

To achieve the best performance out of any system, we need to plan it capacity carefully. Be it the servers, bus lines, or couriers, we need to ensure the system has enough resources to handle the given level of demand.  
This time I'll show you how we can use a mathematical model to make data-driven decisions on the capacity.
Example of food delivery couriers shifts planning.

<!--more-->

## Domain

In the food delivery business, the fleet generally consists of full-time and part-time couriers.
Full-timers provide basic system operations, have strict shifts, and receive a fixed salary. Part-timers, on the other hand, boost the system performance during periods of high demand and earn based on their availability.  

// add an image comparing the two

Our goal is to create an optimal shift schedule for both full-time and part-time couriers, satisfying a set of business constraints.  

## Measure

The first step in improvement is measurement of its current performance.

## Linear Programming to the Rescue

In order to solve this problem, we'll use a mathematical method called [Linear Programming](https://en.wikipedia.org/wiki/Linear_programming) in a way that is supported by the modern programming libraries like [SciPy](https://docs.scipy.org/doc/scipy-0.18.1/reference/generated/scipy.optimize.linprog.html).  

This method requires us to specify:
1. a set of variables.
2. a cost function - a weighted sum of the variables.
3. a set of linear constraints on the variables in a form of linear equations and .  

In return, the automated solver gives us the values of variables that minimise the cost function.  
