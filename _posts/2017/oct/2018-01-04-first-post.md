---
layout: post
date: 2018-01-30
title: "Service-Oriented treatment for Kafkaholism"
description: |
  This is the first post.
keywords:
  - SOA
  - Kafka
  - marketing
categories: first steps
---

This is my **first** blog post on this demo blog..

<!--more-->

2017 was a great year in regards to a number of emerging architectural styles tackling complexity of large software systems.
In this blog post I'd like to discuss the architectural style that continually draws attention to the public.
Microservices hype began to decline and a lot of people understood that it was slightly misleading to move from monolithic to distributed mess.  
Take a look at two pictures from NginX tech blog explaining the path from monolith to microservices:
[Introduction to Microservices](https://www.nginx.com/blog/introduction-to-microservices/)

![Monolith](https://s3.eu-central-1.amazonaws.com/ilyazinkovich-blog-images/NginX-tech-blog-monolith.png)
![Microservices](https://s3.eu-central-1.amazonaws.com/ilyazinkovich-blog-images/NginX-tech-blog-microservices.png)

The whole blog post is about how to move from somewhat looking like a hexagonal architecture to a system of interconnected coupled services. The principle is very simple: for each noun in the former system create a separate service exposing RESTful API.

Everyone who ever tried to implement something like this knows how complicated the whole solution becomes.  
When this mess is already in place we need to find a solution to the newly created problem.  
You open up youtube and search for it.
Hopefully, you find a talk on Four Distributed Systems Architectural Patterns by Tim Berglund and see extremely familiar picture: 

The second post is [here]({% post_url 2017-10-28-second-post %}).
