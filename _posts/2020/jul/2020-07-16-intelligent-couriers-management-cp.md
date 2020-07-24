---
layout: post
date: 2020-07-16
title: "Intelligent Couriers Management"
description: |
keywords:
  - deliveries
  - optimization
  - algorithms
categories: algorithms
urlimage: https://ilyazinkovich-blog-images.s3.eu-central-1.amazonaws.com/2020-07-16-intelligent-couriers-management/icon.png
published: true
---

[Last time](https://bit.ly/37Omcpz) we discussed how to boost the performance of a food delivery service using advanced dispatching algorithms. Despite all the innovation on that front, dispatching algorithms don't perform well when there are just not enough couriers around.  
To satisfy the expected level of demand, we need to hire and manage our couriers intelligently, ensuring that there's always the right number of couriers available all the time.  
In this article, we'll discuss how a universal method from applied maths can help us to solve this problem.  

<!--more-->

## Challenge

Couriers management can be pretty chaotic in the food delivery business. We start forming our fleet with full-time salaried couriers to support core operations. Then we experiment with adding the part-timers to boost the system performance during periods of high demand. As a result, we appear in a situation when our couriers spend only about half of their time delivering orders.  

![alt text](/images/posts/2020-07-16-intelligent-couriers-management/hourly-couriers-engagement-zoom-out.svg?style=centered "Couriers Engagement Zoom Out")  

If we zoom in and check hourly couriers' engagement, we'll see a reason for this - our supply is not elastic enough. Periods of very high engagement are followed by periods when many couriers are idle.  

![alt text](/images/posts/2020-07-16-intelligent-couriers-management/hourly-couriers-engagement-zoom-in.svg?style=centered "Couriers Engagement Zoom In")  

Idle couriers' time is a waste. Shouldn't the couriers spend all their time delivering orders?  
Besides the fact that it's hardly achievable in practice, it's also suboptimal theoretically. In the presence of variable demand, which is a reality of the food delivery business, very high utilization causes a significant growth in delivery time, negatively affecting the customer experience. However, there's always a sweet spot. Let's assume our service can achieve this balance at 80% couriers engagement.  
If we set ourselves an ambitious target to maintain this level throughout the day, the picture will change as follows.  

![alt text](/images/posts/2020-07-16-intelligent-couriers-management/hourly-couriers-engagement-ideal.svg?style=centered "Couriers Engagement Ideal")  

Alright, let's now build a shift schedule for our couriers that conforms to this level of capacity.

## Method Overview

To solve this problem, we'll use a mathematical method called [Linear Programming](https://en.wikipedia.org/wiki/Linear_programming) in a way that is supported by modern programming libraries like [SciPy](https://docs.scipy.org/doc/scipy-0.18.1/reference/generated/scipy.optimize.linprog.html).  

This method requires us to specify:
1. a set of variables with their value ranges.
2. a cost function - a weighted sum of the variables.
3. a set of constraints on the variables in the form of linear equations.  

In return, the automated solver gives us the values of variables that minimize the cost function.  

## Solution

We'll start modeling the full-timers 9-hour shifts.
At first, we need just two variables: one for the number of couriers in the morning shift, another - for the evening shift.
Combining these variables with our capacity requirements for each working hour of a day we'll get the following system of equations:

![alt text](/images/posts/2020-07-16-intelligent-couriers-management/full-timers.svg?style=centered "Full Timers")  

As you can see, there is no solution because the same variable cannot have two distinct values (e.g. Xmorning = 6 and Xmorning = 18).
To proceed with our modeling approach, we need to subtract slack variables, which represent extra capacity in every equation.

![alt text](/images/posts/2020-07-16-intelligent-couriers-management/full-timers-slack.svg?style=centered "Full Timers - Slack")  

Our goal then becomes the minimization of the total number of couriers working hours and excess capacity.
Formulated as a cost function, it will look as follows.

![alt text](/images/posts/2020-07-16-intelligent-couriers-management/full-timers-slack-cost-function.svg?style=centered "Full Timers - Slack, Cost Function")  

The solver consumes the system of equations together with the cost function and gives the following solution for this problem.

![alt text](/images/posts/2020-07-16-intelligent-couriers-management/full-timers-shifts.svg?style=centered "Full Timers Shifts")  

This schedule is far from perfect. The couriers' engagement got worse - 42%, compared to the original 55%.  
But we haven't finished yet, let's add part-timers. We need to add the variables that represent the number of part-time couriers available per hour.  

![alt text](/images/posts/2020-07-16-intelligent-couriers-management/full-timers-slack-part-timers.svg?style=centered "Full Timers - Slack + Part Timers")  

Cost function will include the sum of part-time couriers working hours.  

![alt text](/images/posts/2020-07-16-intelligent-couriers-management/full-timers-slack-part-timers-cost-function.svg?style=centered "Full Timers - Slack + Part Timers, Cost Function")  

In addition to that, we'll specify the constraint on how many part-time couriers we can have per hour. Let's say we can acquire a maximum of 50 part-timers per hour.  
The solver uses this information and gets us the following result.

![alt text](/images/posts/2020-07-16-intelligent-couriers-management/full-timers-part-timers-shifts.svg?style=centered "Full Timers + Part Timers Shifts")  

With this schedule our couriers engagement goes up from 55% to 73%.  
It's a great result on its own, but it's even more important that we've built a whole model that can adjust to new data and new business constraints - the model that grows with the business and maintains its efficiency.  

To access the solver code, visit [my GitHub repo](https://bit.ly/3fLW7KM).
