---
layout: page
title: "Contents"
permalink: contents.html
description: |
  Contents of the entire blog, in chronological order; the page
  is generated automatically and contains a complete list
  of all articles published in this blog
keywords:
  - blog
  - contents
  - software engineering blog
  - ilya zinkovich blog
---

<div id="all">
    {% for post in site.posts %}
    <article itemscope="" itemtype="http://schema.org/BlogPosting" style="margin: 1em 0">
      <meta itemprop="image" content=""/>
      <div itemscope="" itemprop="author" itemtype="http://schema.org/Person">
        <meta itemprop="name" content="Ilya Zinkovich"/>
      </div>
      <h2 itemprop="name headline mainEntityOfPage"><a href="{{post.url}}"><span>{{ post.title }}</span></a></h2>
      <div class="subline">
        <ul>
          <li>{{ post.date | date: "%-d %B %Y" }}</li>
          <li>
            {% capture words %}
            {{ post.content | number_of_words | minus: 180 }}
            {% endcapture %}
            {% unless words contains '-' %}
            {{ words | plus: 180 | divided_by: 180 | plus: 1 | append: ' minutes to read' }}
            {% endunless %}
          </li>
        </ul>
      </div>
    </article>
  {% endfor %}
</div>
