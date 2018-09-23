---
layout: post
date: 2018-09-28
title: "Asynchronous Communication with Legacy System"
description: |
keywords:
  - async
  - legacy
  - distributed systems
categories: distributed systems
urlimage: 
published: false
---

Implementing anything inside a legacy codebase can be painful. 

>"Legacy code. The phrase strikes disgust in the hearts of programmers. Although our first joy of programming may have been intense, the misery of dealing with legacy code is often sufficient to extinguish that flame."  
>[Michael Feathers, Working Effectively with Legacy Code](https://www.amazon.de/Working-Effectively-Legacy-Robert-Martin/dp/0131177052).  

That's why we, developers, naturally tend to implement new functionality outside of it. With the rise of microservices, this tendency escalated to a whole new level. Anytime we want to implement something new, we think of creating a new web service. Applying this approach regularly, we add more moving parts. Losing control over this process, we risk creating a highly unreliable system.  

We've already discussed how to sustain [Reliable Services Communication]({% post_url 2018/jun/2018-06-02-reliable-services-communication %}) for relatively new services.
But what if we're stuck with a legacy system that everyone is scared to change for some valid reasons?

<!--more-->

## What we got

Suppose we work in e-commerce domain. We have a huge legacy codebase, and we've identified a Product Catalog sub-domain that we can extract into a separate service to scale it independently and leverage continuous delivery for faster customer feedback.  
As usual, there appears some part in our system that doesn't belong to Product Catalog sub-domain but still requires its data - and this time it's Billing. The only thing the Billing part wants to know is the price per product id.
How can we extract Product Catalog and make this information available for Billing?

## Exposing HTTP API

Exposing Product Catalog data via HTTP API is a straightforward, well-known approach.

![alt text](https://bit.ly/2O2SFRb?style=centered "sync legacy")

However, it's not a good choice for the following reasons:
1. Both services should be up and running at the same time for Billing to work. Of course, it's not always the case in distributed systems. For Billing to work correctly, we need to handle situations when Product Catalog is down. It will inevitably complicate Billing logic.
2. Billing response time increases due to additional remote call and translation of data from the Product Catalog domain to Billing.
3. Billing adds load on Product Catalog. This factor is often overlooked. While Product Catalog should be busy answering end-user queries, it should reserve enough capacity to serve Billing requests. And there is no guarantee that other parts of the system won't start using this API too.
4. Billing becomes more complicated to test. Integration testing for services making HTTP calls is complex. Mocked HTTP clients or even whole application parts don't contribute to the confidence that we expect from this kind of testing.

## Asynchronous Communication

The answer is to communicate Product Catalog data back to the legacy system asynchronously. And here is the most non-invasive way to implement this.

![alt text](https://bit.ly/2OM4wAz?style=centered "async legacy")

When Product Catalog changes product price, it publishes an event containing the latest price for a product id.
Events get persisted in some reliable messaging system like AWS Kinesis or Kafka.
Finally, a simple component like AWS Lambda or Kafka Stream processor handles each event, translates it into Billing context and persists it in a database that is already used by a Billing part.  

Someone might argue that this is a bit complex design for such a simple problem. However, without the problems listed above it pays off very quickly as the system evolves. If messaging infrastructure is already in place, the only additional component is event processor. It logically belongs to the Billing context and can become a part of a newly extracted Billing service without any heavy-lifting.
