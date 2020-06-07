---
layout: post
date: 2020-06-01
title: "Evolution of Food Delivery Dispatching"
description: |
keywords:
  - deliveries
  - dispatching
  - algorithms
categories: algorithms
urlimage: 
published: false
---

In this article, I'd like to show how the domain model of the food delivery dispatching evolves with the business. And which classic optimization problems are at the core of the service we all enjoyed a lot during the quarantine.  

<!--more-->

## Level 0. Individual Assignment

Imagine we are the new delivery startup. We've just started our business and decided to attract customers by offering low delivery times.  
We need to get the basics right, so we start with a straightforward dispatching algorithm - when the customer places the order, we assign the closest idle courier to it.

![alt text](https://ilyazinkovich-blog-images.s3.eu-central-1.amazonaws.com/2020-06-07-deliveries-dispatching-evolution/level-0.svg?style=centered "Level 0")

## Level 1. Collective Assignment

As we progress, we start seeing that couriers waste a lot of their time at restaurants waiting for the order to be prepared. We can ask restaurants to provide us approximate food preparation time when they accept the order (or predict it with our groundbreaking machine learning model) and then leverage this information to make dispatching decisions that minimize waiting time.  
But as we know that we have some time before the courier needs to arrive at the restaurant, we can do more than that. Imagine two orders dispatched at nearly the same time. Couriers' expected arrival time is on the diagram along with the time when the order will be ready at the respective restaurant. Which courier would you choose for which order?  

![alt text](https://ilyazinkovich-blog-images.s3.eu-central-1.amazonaws.com/2020-06-07-deliveries-dispatching-evolution/concurrent-dispatch.svg?style=centered "Concurrent Dispatch")

In this situation, two orders are competing couriers. The first getting assigned will win, the other will lose. But will our business win as a result? Sometimes it will, other times - it won't.  
Instead of relying on pure luck, we can use some portion of time while the orders are prepared to collect them and then run a fair algorithm that will judge which assignment is economically better.  
Fortunately, we have a mathematical model for that called [Assignment Problem](https://en.wikipedia.org/wiki/Assignment_problem) with a reliable [Hungarian Algorithm](https://en.wikipedia.org/wiki/Hungarian_algorithm) that solves it. The only thing we need is to define the criteria for the algorithm to compare the alternatives. In our case, we can say that each assignment incurrs the waiting cost (if the courier arrives too early) and the delay cost (if the captain arrives too late). The algorithm will use this information and dispatch the orders minimising the total cost.  

![alt text](https://ilyazinkovich-blog-images.s3.eu-central-1.amazonaws.com/2020-06-07-deliveries-dispatching-evolution/assignment-problem.svg?style=centered "Assignment Problem")

## Level 2. Pooling

As our business grows, we start experiencing "supply crunch" - every breakfast, lunch and dinner we get more orders that our couriers can handle and we start rejecting the orders. (show it on a diagram - idle couriers/minute vs number of orders) We've already tried optimal shifts planning and employed the maximum number of couriers that makes economical sense. In order to maximise the efficiency of our supply at the cost of potential increase in the delivery time, we deide to implement pooling of orders - we want to be able to assign multiple order to a single courier at once.
Can the Hungarian algorithm handle this? - Yes, but we should change the definition of a "task" in a classical algorithm.
We'll group the orders before solving the Assignment Problem, including the orders that the couriers are currently assigned to if this makes sense (e.g. when the courier is close to the drop-off, or is on the way to the same restaurant as the non-assigned order in a batch). The chain of orders must pass the artificial constraints that we introduce to control the delivery time. And the chain will be assigned as a single unit to the courier in the Hungarian algorithm.
And if the algorithms chooses the in-ride captain, we can even delay this assignment an wait for some time as the waiting will not reduce the efficiency of our supply, but will save us some time to find more idle couriers in the next batch.

## Level 3. Mixed-Integer Programming

You don't consider assigning order 1 vs assigning order 1 + order 2.
For this we should as a Mixed Integer Problem, more precisely - the vehicle routing problem (VRP), which is described in details in the DoorDash blog: https://doordash.engineering/2020/02/28/next-generation-optimization-for-dasher-dispatch-at-doordash/amp
One important point here is that to solve this problem correctly and efficiently, we absolutely must use a commercial Optimization Solver like Gurobi, XPress, CPLEX and the like. Fully utilise the free trial to measure the profit to the price of the commercial package.

Icons made by <a href="https://www.flaticon.com/authors/ultimatearm" title="ultimatearm">ultimatearm</a> from <a href="https://www.flaticon.com/" title="Flaticon"> www.flaticon.com</a>