---
layout: post
date: 2018-03-28
title: "DDD approach to TinyURL system design interview problem"
description: |
  DDD approach to TinyURL system design interview problem.
keywords:
  - DDD
  - domain-driven design
  - system design
categories: system design
---

System design is the most tackling topic of technical interviews and my personal choice to get the most valuable insight about the candidate. Although there is no right answer to the questions in this area, the answer shows developer experience, architectural skills and ability to make strategic decisions. Let's dive into the TinyURL service design problem and explore how Domain-Driven Design can be a valuable foundation for our solution.

<!--more-->

TinyURL service should generate a short version of a given URL and return the original URL given its shortened version. 

Let's event storm our requirements.

**TinyURL service should generate a short version of a given URL...**  
Short URL generation drives the rest of our system behavior. It, therefore, deserves a dedicated `ShortURLGenerated` event and `GenerateShortURL` command leading to the event publishing.  

**... return the original URL given its shortened version.**  
This functionality derives from the application state and can be modeled as a GetOriginalURL query rather than as an event.  

Grouping related functionality we get two aggregates:
* `ShortURLGenerator` that accepts the `GenerateShortURL` command and publishes the `ShortURLGenerated` event.
* `URLCache` that subscribes to the `ShortURLGenerated` and accepts `GetOriginalURL` query.

The second post is [here]({% post_url 2017-10-28-second-post %}).
