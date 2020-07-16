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

[Last time](https://bit.ly/37Omcpz) we discussed how to boost the performance of a food delivery service by the use of advanced dispatching algorithms. Despite all the innovation we can do on that front, dispatching algorithms don't perform well when there're simply not enough couriers around.  
To meet the expected level of demand we need to hire and manage our couriers accordingly, ensuring that there's always the right number of couriers at the times when we need them.  
This problem is not unique and is relevant in various contexts: from the bus lines scheduling to the proactive scaling of the cloud servers.  
Let's dig into one possible solution.  

<!--more-->

## Challenge

Couriers hiring and management is pretty chaotic in the new food delivery startup. We start forming our fleet out of full-time salaried couriers to support basic operations and then experiment with adding the part-timers to boost the system performance during periods of high demand. After applying all of our efforts we appear in the following situtation - couriers spend only about 50% of their time delivering orders.  

![alt text](/images/posts/2020-07-16-shifts-planning--hourly-couriers-engagement-zoom-out.svg?style=centered "Couriers Engagement Zoom Out")  

If we zoom in and check hourly couriers' engagement, we'll see a clear reason for this - our supply is not elastic enough. Periods of very high engagement alternate with very low.  

![alt text](/images/posts/2020-07-16-shifts-planning--hourly-couriers-engagement-zoom-in.svg?style=centered "Couriers Engagement Zoom In")  

Given this information, a natural question arises: what is the ideal couriers engagement? Shouldn't the couriers be busy all the time?
Simply put, no. In the presence of variable demand (which is a reality of the food delivery startups) high utilisation causes significant growth in delivery time which directly affects customer experience.  
However, there's always a sweet spot which balances efficiency with customer experience. Let's assume for our current system this balance is achieved at 80% couriers engagement. As we set ourselves an ambitious target to maintain it throughout the day the picture should change as follows.  

![alt text](/images/posts/2020-07-16-shifts-planning--hourly-couriers-engagement-ideal.svg?style=centered "Couriers Engagement Ideal")  

Alright, let's now build a shifts schedule for our couriers that conforms to this level of capacity.

## Linear Programming Overview

In order to solve this problem, we'll use a mathematical method called [Linear Programming](https://en.wikipedia.org/wiki/Linear_programming) in a way that is supported by the modern programming libraries like [SciPy](https://docs.scipy.org/doc/scipy-0.18.1/reference/generated/scipy.optimize.linprog.html).  

This method requires us to specify:
1. a set of variables with their value ranges.
2. a cost function - a weighted sum of the variables.
3. a set of constraints on the variables in the form of linear equations.  

In return, the automated solver gives us the values of variables that minimise the cost function.  

## Solution

Let's start modelling the full-timers shifts.
At first, we need two variables: one for the number of couriers in the morning shift, another - for the evening shift.
Combining these variables with our capacity requirements for each hour of a day we'll get the following system of equations:

Xmorning = 20 (required number of couriers at 7:00AM)
Xmorning = 30 (8:00AM)
...
Xmorning + Xevening = 100 (2:00PM)
...
Xevening = 40 (9:00PM)
Xevening = 30 (10:00PM)

As you can see, this system doesn't have a solution, because the same variable cannot have two values (e.g. Xmorning = 20 and Xmorning = 30).
In order to proceed with our approach, we need to add slack variables to every equation.

Xmorning + Xslack7 = 20 (7:00AM)
Xmorning + Xslack8 = 30 (8:00AM)
...
Xmorning + Xevening + Xslack14 = 100 (2:00PM)
...
Xevening + Xslack21 = 40 (9:00PM)
Xevening + Xslack22 = 30 (10:00PM)

These variables will determine the number of couriers above the required level of capacity.  

Our goal then becomes the minimisation of the total number of couriers working hours a day along with the minimisation of excess capacity.
Formulated as a cost function it will look as follows: 9 * Xmorning + 9 * Xevening + sum(Xslack(i))

The solver gives the following solution for this problem:

// only full-timers

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
