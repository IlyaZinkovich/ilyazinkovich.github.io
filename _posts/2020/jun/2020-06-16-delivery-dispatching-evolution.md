---
layout: post
date: 2020-06-16
title: "Evolution of Food Delivery Dispatching"
description: |
keywords:
  - deliveries
  - dispatching
  - algorithms
categories: algorithms
urlimage: https://ilyazinkovich-blog-images.s3.eu-central-1.amazonaws.com/2020-06-07-deliveries-dispatching-evolution/preview.png
published: true
---

After quite some time of working on dispatching at [Careem](https://www.careem.com), I'd like to show how dispatching evolves with the business and which classic mathematical problems are at the core of the service we all enjoyed during the lockdown.  
Imagine we develop a dispatching service for the new food delivery startup.  

<!--more-->

## Level 0. Individual Assignment

We've just started our business and decided to attract customers by offering low delivery time.  
We need to get the basics right, so we start with a straightforward dispatching algorithm - when the customer places the order, we assign the closest idle courier to it.

![alt text](https://ilyazinkovich-blog-images.s3.eu-central-1.amazonaws.com/2020-06-07-deliveries-dispatching-evolution/level-0.svg?style=centered "Level 0")

## Level 1. Batch Assignment

As we progress, we notice that couriers waste a lot of their time at restaurants waiting for the order to be prepared. We can ask restaurants to provide us approximate food preparation time when they accept the order (or predict it with a machine learning model) and leverage this information in dispatching decisions to minimize waiting time, increasing our couriers' efficiency.  
But as we have some time before the courier needs to arrive at the restaurant, we can do more.  
Imagine two orders dispatched at nearly the same time. Couriers' expected arrival time is on the diagram along with the time when the order will be ready at the respective restaurant. Which courier would you choose for which order?  

![alt text](https://ilyazinkovich-blog-images.s3.eu-central-1.amazonaws.com/2020-06-07-deliveries-dispatching-evolution/concurrent-dispatch.svg?style=centered "Concurrent Dispatch")

In this situation, two orders are competing for couriers. The first assigned will win, the other will lose. But will our business win as a result? Sometimes it will, other times - it won't.  
Instead of relying on pure luck, we can use some portion of food preparation time to batch the orders and then run a fair algorithm that will choose the economically best assignment.  
Fortunately, we have a mathematical model for that called [Assignment Problem](https://en.wikipedia.org/wiki/Assignment_problem) with a reliable [Hungarian Algorithm](https://en.wikipedia.org/wiki/Hungarian_algorithm) that solves it. The only thing we need is to define the criteria for the algorithm to compare the alternatives, known as the cost function. For instance, we can say that each assignment incurrs the waiting cost (if the courier arrives too early) and the delay cost (if the captain arrives too late). The algorithm will use this information and dispatch the orders minimising the total cost.  

![alt text](/images/posts/2020-06-07-deliveries-dispatching-evolution--assignment-problem.svg?style=centered "Assignment Problem")

## Level 2. Pooling

As our service becomes more popular, we start experiencing periods when the demand for our service significantly outstrips the economically viable level of supply, which leads to an increased number of delays and cancellations. The difference between the number of incoming orders and the number of available couriers looks as follows.  

![alt text](/images/posts/2020-06-07-deliveries-dispatching-evolution--supply-demand.svg?style=centered "Supply Demand Mismatch")

How can we solve this problem? - Do more with less. Or, more precisely, group orders by restaurant and let a single courier deliver multiple orders at a time.  
The problem is that the Hungarian Algorithm does only 1-1 assignments. To prove that our idea works without a major rewrite, we can hack it a little - generate multi-order routes (with some greedy/brute-force algorithm) and then assign routes as a unit of work.  

![alt text](/images/posts/2020-06-07-deliveries-dispatching-evolution--basic-pooling.svg?style=centered "Supply Demand Mismatch")

## Level X. Heavy Machinery

Although a combination of the route generator and the Hungarian algorithm solver is a viable solution to start pooling the orders, it has a few major drawbacks:
- greedy route generation can be suboptimal, brute-force - too slow.
- once we generated the route with 2 orders, we aren't able to assign each of the orders separately, which can be a good alternative on its own.
- route generator and Hungarian algorithm solver are hard to align - as both algorithms work separately they are very likely to optimise for different objectives.  

The solution is to unite these two algorithms under a single consistent model - [the Vehicle Routing Problem](https://en.wikipedia.org/wiki/Vehicle_routing_problem), which avoids the issues outlined above by design and is described in great detail on [the DoorDash blog](https://doordash.engineering/2020/02/28/next-generation-optimization-for-dasher-dispatch-at-doordash/amp/).  
One important point in the DoorDash article is that to solve this problem correctly and efficiently, we should use a commercial Optimization Solver like [Gurobi](https://www.gurobi.com), [CPLEX](https://www.ibm.com/analytics/cplex-optimizer), or [XPress](https://www.fico.com/en/products/fico-xpress-optimization).  
Mixing accurate data with tailored cost functions plugged into these high-quality optimization solvers, we can achieve high-quality dispatching, which ensures that supply meets demand in the best economical way.  

With all these batteries included, our primary effort becomes the development of exceptional experiences on top of this powerful dispatching engine at the heart of the delivery business.  

<p style="font-size: 10px">Icons made by <a href="https://www.flaticon.com/authors/ultimatearm" title="ultimatearm">ultimatearm</a> from <a href="https://www.flaticon.com/" title="Flaticon"> www.flaticon.com</a></p>
