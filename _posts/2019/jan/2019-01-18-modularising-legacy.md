---
layout: post
date: 2019-01-18
title: Modularising Legacy Systems
description: |
keywords:
  - modular design
categories: modular design
urlimage: 
published: false
---

// Have you ever thought why legacy codebases are so hard to maintain?
I often find myself in a codebase where the parts are highly interconnected, and each of them has some meaning only in combination with others. This issue is known as the lack of local reasoning. You know you have it if even a small isolated fix requires the knowledge of the whole system or its significant part. Over time I developed a technique that enables the local reasoning and helps to modularise even the messiest codebases. And I'd like to share it with you.


// Only the bravest create test-implementations for huge interfaces. The others use the mocking library of choice and.

<!--more-->
