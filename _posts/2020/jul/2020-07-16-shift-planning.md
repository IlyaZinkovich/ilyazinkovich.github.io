---
layout: post
date: 2020-07-01
title: "Data-Driven Courier Shift Planning"
description: |
keywords:
  - deliveries
  - optimization
  - algorithms
categories: algorithms
urlimage: 
published: false
---

[Last time](https://bit.ly/37Omcpz) we discussed how to boost the performance of a food delivery service by the use of advanced dispatching algorithms. Despite all the innovation we can do on that front, dispatching algorithms don't perform well when there's simply not enough couriers around.  
In order to meet the expected level of demand, we need to plan our couriers' shifts carefully.  
This problem is not unique and was solved already in various contexts: from the bus lines scheduling to the proactive scaling of the cloud servers.  

<!--more-->

## Domain

In the food delivery business, the fleet generally consists of full-time and part-time couriers.
Full-timers provide basic system operations, have strict shifts, and receive a fixed salary. Part-timers, on the other hand, boost the system performance during periods of high demand and earn based on their availability.  

// add an image comparing the two

Our goal is to create an optimal shift schedule for both full-time and part-time couriers, satisfying a set of business constraints.  

## Measure

The first step in improvement is measurement of the current performance.
If we collect telemetry from couriers' devices and divide their available time into busy (when they are delivering the order) and idle (when they are waiting for the new order) we'll see the following picture:

// add an image of utilisation

Hm, our average utilisation (total busy hours / total available hours) is only about 50%. Shouldn't our couriers be busy all the time?  
Simply put, no. In the presence of variable demand (which is a natural quality of the food delivery business) high utilisation causes significant growth in delivery time which directly affects customer experience.  
However, there's always a sweet spot which balances efficiency with customer experience. Let's say for our system the optimal ulitisation is 80%. And we can set ourselves an ambitious target to maintain it throughout the day.

// add an image (plot) of utilisation before and after

Alright, let's now build a shifts schedule for our couriers that conforms to this level of capacity.

## Linear Programming

In order to solve this problem, we'll use a mathematical method called [Linear Programming](https://en.wikipedia.org/wiki/Linear_programming) in a way that is supported by the modern programming libraries like [SciPy](https://docs.scipy.org/doc/scipy-0.18.1/reference/generated/scipy.optimize.linprog.html).  

This method requires us to specify:
1. a set of variables with their value ranges.
2. a cost function - a weighted sum of the variables.
3. a set of constraints on the variables in the form of linear equations.  

In return, the automated solver gives us the values of variables that minimise the cost function.  

## Modelling

// only full-timers infeasible
// only part-timers feasible
// add part-timers

The variables will be the number of captains in a shift.
We'll have two 9-hour full-time shifts: one that starts in the morning and another one that ends in the evening.
In addition to that we'll have one-hour shifts for part-timers per every working hour of our service.
The cost function will calculate the total number of working hours.
c(x) = 9Xmorning + 9Xevening + sum(Xi, i in (7, 22))
Our constraints will make sure that the number of couriers every hour is equal to the optimal.
For example, for the 7th hour of a day the number of couriers will be Xmorning + X7 = 20.
While for the 14th hour of a day it will be Xmorning + X14 + Xevening = 100.
The solver will output the following result:

// add an image which shows that we need to hire mostly part-timers all the time

We perfectly matched the requirements! But is it really possible to acquire 90 part-time couriers at peak for just one hour? You can ask your courier acquisition team and get an answer like: 90 - no, but half of that sounds reasonable.  
How can we include this information into our model?
We can specify a constraint for our variables, saying that Xi < 40 for i in (7, 22).

If we run the solver again, we'll get the following output:
'The algorithm terminated successfully and determined that the problem is infeasible.'

And here is why. Consider the following equations:
Xmorning + X7 = 20
Xmorning + X10 = 70

X10 - X7 = 50

But as we agreed there is no way the difference between two hourly part-time shifts can exceed 50, becase both of them can consist max of 40 couriers.

We need to modify the model a bit to achieve sane results.
Let's add the "slack" variables for every working hour that will mean the value by which we can diverge from the optimal capacity.

The cost function will then look as follows:
c(x) = 9Xmorning + 9Xevening + sum(Xi, i in (7, 22)) + sum(Xslacki, i in (7, 22))
And the constraints will include slack as well:
Xmorning - Xs7 + X7 = 20
and so on.

Let's run the solver again.
Here is hour optimal shifts schedule:

// add an image with the optimal shifts schedule
