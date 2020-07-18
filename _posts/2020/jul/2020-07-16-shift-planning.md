---
layout: post
date: 2020-07-16
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

[Last time](https://bit.ly/37Omcpz) we discussed how to boost the performance of a food delivery service by the use of advanced dispatching algorithms. Despite all the innovation we can do on that front, dispatching algorithms don't perform well when there're simply not enough couriers around.  
To satisfy the expected level of demand we need to hire and manage our couriers intelligently, ensuring that there's always the right number of couriers at all times.  
In this article, we'll discuss how a universal method from applied maths can help us to solve this problem.  

<!--more-->

## Challenge

Couriers hiring and management can be pretty chaotic in the food delivery business. We start forming our fleet with full-time salaried couriers to support basic operations and then experiment with adding the part-timers to boost the system performance during periods of high demand. After applying all of our efforts we appear in the following situtation - couriers spend only about 50% of their time delivering orders.  

![alt text](/images/posts/2020-07-16-shifts-planning/hourly-couriers-engagement-zoom-out.svg?style=centered "Couriers Engagement Zoom Out")  

If we zoom in and check hourly couriers' engagement, we'll see a clear reason for this - our supply is not elastic enough. Periods of very high engagement alternate with very low.  

![alt text](/images/posts/2020-07-16-shifts-planning/hourly-couriers-engagement-zoom-in.svg?style=centered "Couriers Engagement Zoom In")  

Hm, shouldn't the couriers be busy all the time?  
Simply put, no. In the presence of variable demand (which is a reality of the food delivery business) high utilisation causes significant growth in delivery time which directly affects customer experience.  
However, there's always a sweet spot. Let's assume for our current system the balance is achieved at 80% couriers engagement. If we set ourselves an ambitious target to maintain this level throughout the day the picture should change as follows.  

![alt text](/images/posts/2020-07-16-shifts-planning/hourly-couriers-engagement-ideal.svg?style=centered "Couriers Engagement Ideal")  

Alright, let's now build a shifts schedule for our couriers that conforms to this level of capacity.

## Method Overview

In order to solve this problem, we'll use a mathematical method called [Linear Programming](https://en.wikipedia.org/wiki/Linear_programming) in a way that is supported by the modern programming libraries like [SciPy](https://docs.scipy.org/doc/scipy-0.18.1/reference/generated/scipy.optimize.linprog.html).  

This method requires us to specify:
1. a set of variables with their value ranges.
2. a cost function - a weighted sum of the variables.
3. a set of constraints on the variables in the form of linear equations.  

In return, the automated solver gives us the values of variables that minimise the cost function.  

## Solution

We'll start modelling the full-timers shifts.
At first, we need just two variables: one for the number of couriers in the morning shift, another - for the evening shift.
Combining these variables with our capacity requirements for each working hour of a day we'll get the following system of equations:

![alt text](/images/posts/2020-07-16-shifts-planning/full-timers.svg?style=centered "Full Timers")  

As you can see, there is no solution because the same variable cannot have two distinct values (e.g. Xmorning = 20 and Xmorning = 30).
In order to proceed with our approach, we need to subtract slack variables which represent extra capacity in every equation.

![alt text](/images/posts/2020-07-16-shifts-planning/full-timers-slack.svg?style=centered "Full Timers - Slack")  

Our goal then becomes the minimisation of the total number of couriers working hours a day along with the minimisation of excess capacity.
Formulated as a cost function it will look as follows.

![alt text](/images/posts/2020-07-16-shifts-planning/full-timers-slack-cost-function.svg?style=centered "Full Timers - Slack, Cost Function")  

The solver consumes the system of equations together with the cost function and gives the following solution for this problem.

![alt text](/images/posts/2020-07-16-shifts-planning/full-timers-shifts.svg?style=centered "Full Timers Shifts")  

Adding part timers.

![alt text](/images/posts/2020-07-16-shifts-planning/full-timers-slack-part-timers.svg?style=centered "Full Timers - Slack + Part Timers")  

Cost function.

![alt text](/images/posts/2020-07-16-shifts-planning/full-timers-slack-part-timers-cost-function.svg?style=centered "Full Timers - Slack + Part Timers, Cost Function")  

Result.

![alt text](/images/posts/2020-07-16-shifts-planning/full-timers-part-timers-shifts.svg?style=centered "Full Timers + Part Timers Shifts")  

We perfectly matched the requirements! But is it really possible to acquire 90 part-time couriers at peak for just one hour? You can ask your courier acquisition team and get an answer like: 90 - no, but half of that sounds reasonable.  
How can we include this information into our model?
We can specify a constraint for our variables, saying that Xi < 40 for i in (7, 22).
