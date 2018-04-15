---
layout: post
date: 2018-04-15
title: "Data Mining Git for Design Insights"
description: |
keywords:
  - code analysis
  - coupling
  - tools
categories: code analysis
---

Data Science is a trendy topic nowadays. We analyze customers behavior when they are browsing an e-commerce website to improve user experience, apply fraud detection mechanisms to prevent malicious transactions and use image recognition to know [what's a hot dog and what is not](https://www.youtube.com/watch?time_continue=63&v=ACmydtFDTGs).  
At the same time, there are not too many data-powered instruments that would help us, software engineers, improve our experience of writing high-quality code.  
Git, in its turn, is not only a must-have tool in our day-to-day job but also a source of valuable data hiding the insights about both source code and people who develop it.

<!--more-->

Recently I watched the talk by Greg Young about [how to get productive on a project in 24h](https://youtu.be/KaLROwp-VDY?t=1m24s).
He proposed to measure a number of changes we make to files over time and argues that in a wealthy code base we tend to make intensive changes at the beginning and minor less frequent changes like a bug fix further.
On the other hand, when we deal with a low-quality code, we go back to the same files again and again over the course of a project and make significant changes.  
To test this assumption I've implemented this tool and analyzed the code of my favorite plugin - [Sonar Lint"](https://plugins.jetbrains.com/plugin/7973-sonarlint).

![alt text](https://bit.ly/2qxeA6g?style=centered "changes frequency analysis results")

The table view contains information about the file name and a number of concurrent changes for this file.
In this example, I consider the changes made within two days to be concurrent. 
The graph below shows commits made to a selected file. Horizontal scale depicts a number of days since commit, vertical - a number of days left since the previous commit.
The higher the points are on the graph - the more stable is the code. 
`SonarLintUtils` is an example of a problematic class. Developers made a large number of regular changes to it over more than two years.  

This situation is familiar to most of us. It happens when there are some missing design concepts, and somebody without proper thinking or under a high pressure decides to put arbitrary logic to `WhateverUtils` class laying a foundation for a new junkyard on a project. 

Another thing that interests me is how changes are related to each other.  
When I fix some specific part of a legacy project, I usually find myself making changes in a whole set of places (that sometimes even seem unrelated to a problem) just to make the code compile.  

I've built [another tool](https://github.com/IlyaZinkovich/git-data-mining) to cluster the files that we change together within one commit. 
It identifies which files changed in each commit, creates a node in a graph for each file and creates edges between files changed within the same commit. Then we can query the graph and answer how many dependencies each class has.

Analysis of the `SonarLintUtils` class revealed dependencies on the following set of classes:
- `IssueProcessor`
- `SonarLintIssuesPanel`
- `SonarQubeServerMgmtPanel`
- `SonarLintProjectSettings`
- `SonarLintProjectSettingsPanel`

Of course, it's developer's job to decide if it's right or wrong, but having tools to support these decisions makes our job more productive, leaving routine to machines. 
