---
layout: post
date: 2020-06-01
title: "Evolution of Food Delivery Dispatching"
description: |
keywords:
  - deliveries
  - dispatching
  - domain-driven design
categories: domain-driven design
urlimage: 
published: false
---

In this article, I'd like to show how the domain model of the deliveries assignment evolves with the business.
Classic optimization problems at the core of the service we all enjoyed during the quarantine.

<!--more-->

## Level 0. Individual Assignment

We've just started our business and choose to attract the new customers by offering lower delivery times compared to our competitors. The number of orders is low.
We just want to get the basics right and therefore start with the simplest dispatching algorithm.
When the order is placed in the system we assign the closest idle courier to it.

Search for couriers -> Rank them by ETA -> Assign the best.

## Level 1. Batch Assignment

As we get more and more orders, we start to question whether the algorithm makes the right decisions.
We know more about food preparation time. Since the food takes time to prepare, what if we take our time to collect more orders and assign them collectively, minimising the wait time at restaurant.
Hm, sounds like a classic "Assignment Problem" - given N tasks and M workers make optimal 1-1 assignments. (minimise the cost of assignment)
We play with the cost function that balance the efficiency with customer experience.
Search for couriers -> Rank them by Cost Function -> Solve the Assignment Problem with Hungarian algorithm.

## Level 2. Pooling Workaround

As our business grows, we start experiencing "supply crunch" - every breakfast, lunch and dinner we get more orders that our couriers can handle and we start rejecting the orders. (show it on a diagram - idle couriers/minute vs number of orders) We've already tried optimal shifts planning and employed the maximum number of couriers that makes economical sense. In order to maximise the efficiency of our supply at the cost of potential increase in the delivery time, we deide to implement pooling of orders - we want to be able to assign multiple order to a single courier at once.
Can the Hungarian algorithm handle this? - Yes, but we should change the definition of a "task" in a classical algorithm.
We'll group the orders before solving the Assignment Problem, including the orders that the couriers are currently assigned to if this makes sense (e.g. when the courier is close to the drop-off, or is on the way to the same restaurant as the non-assigned order in a batch). The chain of orders must pass the artificial constraints that we introduce to control the delivery time. And the chain will be assigned as a single unit to the courier in the Hungarian algorithm.
And if the algorithms chooses the in-ride captain, we can even delay this assignment an wait for some time as the waiting will not reduce the efficiency of our supply, but will save us some time to find more idle couriers in the next batch.

## Level 3. Mixed-Integer Programming

You don't consider assigning order 1 vs assigning order 1 + order 2.
For this we should as a Mixed Integer Problem, more precisely - the vehicle routing problem (VRP), which is described in details in the DoorDash blog: https://doordash.engineering/2020/02/28/next-generation-optimization-for-dasher-dispatch-at-doordash/amp
One important point here is that to solve this problem correctly and efficiently, we absolutely must use a commercial Optimization Solver like Gurobi, XPress, CPLEX and the like. Fully utilise the free trial to measure the profit to the price of the commercial package.

Icons made by <a href="https://www.flaticon.com/authors/ultimatearm" title="ultimatearm">ultimatearm</a> from <a href="https://www.flaticon.com/" title="Flaticon"> www.flaticon.com</a>