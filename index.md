---
layout: default
title: Home
---

*Author: Derek Watson*
<br/>

# Azure AZ-204: Developer Associate certification blog

**Welcome!** This blog documents my preparation for the AZ-204 (Azure Developer Associate) certification. Follow along as I work through each module in the certification syllabus and share my learning journey. This blog was created as part of my semester project for the University of Colorado, Boulder's CSPB 3112: Professional Development in Computer Science 1 credit course.

## Posts

{% for post in site.posts %}
- [{{ post.title }}]({{ post.url | relative_url }})
{% endfor %}
