---
layout: default
title: Home
---

*Author: Derek Watson*
# Welcome to my AZ-204: Developer Associate certification blog

This blog documents my preparation for the AZ-204 (Azure Developer Associate) certification. Follow along as I work through each module in the certification syllabus and share my learning journey.

## Posts

{% for post in site.posts %}
- [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}
