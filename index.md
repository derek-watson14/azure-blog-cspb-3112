---
layout: default
title: Home
---

# Welcome to my AZ-204 Certification Blog
*Author: Derek Watson*

This blog documents my preparation for the AZ-204 (Azure Developer Associate) certification. Follow along as I work through each module in the certification syllabus and share my learning journey.

## Blog Posts
Here are the posts published so far:

{% for post in site.posts %}
- [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}
