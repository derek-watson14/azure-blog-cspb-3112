---
layout: post
title: "Implement caching for solutions"
date: 2025-04-30
---

**LinkedIn Learning Course Link: [Microsoft Azure Developer Associate (AZ-204) Cert Prep by Microsoft Press](https://www.linkedin.com/learning/microsoft-azure-developer-associate-az-204-cert-prep-by-microsoft-press/)**

## Lesson 8: Implement caching for solutions

*Methodology: Watch each video in the lesson and take notes, summarize learning in "post-lesson review" section when done.*

## Post-lesson review
Interesting course, mostly the part about Redis because I was already familair with Azure CDN. I didnt realize how easy Redis was to use, might be useful to implement in some cases in the future.

<hr/>

### Video 1: Learning Objectives

The learning objectives for this lesson are:
- Configure cache and expiration policies for Azure Cache for Redis
- Implement secure and optimized cache patterns including data sizing, connections, encryptions and expiration
- Implement Azure CDN endpoints and profiles

<hr/>

### Video 2: Configure cache and expiration policies for Azure Cache for Redis
Azure Cache for Redis provides in memory data store that is based on redis. Low latency, high throughput, versatile use cases.

It is an Azure Managed Service (handles backups, availability, integrates with other services etc). Can use Redis open source or Redis enterprise.

**Scenarios**
- Data cache (Cache-aside pattern, or only needed data)
- Content caching
- Session store

**SKUs (stock keeping unit)**
- Basic: Single VM, no SLA, cheap, good for development or non-critical workloads
- Standard: Sufficient for production, has 2 VMs replicated configuration
- Premium: More powerful VM with higher throughput, lower latency
- Enterprise: Doesnt use open source, comes with Redis Enterprise module support, even more powerful than premiun
- Enterprise Flash: Extends to use non volatile memory, good for very large data sets

**Epriation policies**
- Absolute epiration: set expiration absolutely
- Sliding expiration: epiration is extended on each access (not absolute)

**Configuration**
Set, sub, rg, name, location, SKU, and cache size in the portal.

<hr/>

### Video 3: Implement secure and optimized cache patterns including data sizing, connections, encryptions and expiration

Main concepts:
- Choose the right size (tier) for the cache
- To do so, need to consider your features and select the tier based on what you need and then size based on how large it is
- Works best with more keys and smaller values
- Can use clustering for key distribution
- Redis pipelining makes efficient use of the network
- Choose the correct version when creating the cache
- Put the client and cache in the same region to maimize performance
- Must use TLS encryption (1.2 should be used as older are deprecated)

**Demo**
- use StackEchange.Redis in .NET
- Need to connect with:
  - Access key (Host Name, Port) or entra ID which can leverage RBAC

- Go to NuGet get the StackExchange package
- Can use console to interact with cache in portal
- Authentication: Use entra authentication or access keys (automatically enabled)
  - Can get AKs in azure portal for the resource
  - Can assign entra roles in Access Control (AIM blade) 
- Important to define TLS version in Advanced settings
- Data access configuration have RBAC roles
- Scale blade allows you to scale cache size and tier

- Create console application with `dotnet new console --name <name>`
- `dotnet add package StackEchange`
- Program.cs:
```C#
using StackExchange.Redis;
using System.Text.Json;

// Set connection string and prefi
// create connection to redis cache using RedisConnection class
// use db.StringGetAsync(key) or db.StringSetAsync(key, value) to get and set values in the cache
// To store more complex objects can use JSON serializer on the value
```
<hr/>

### Video 4: Implement Azure CDN endpoints and profiles

**What is a CDN**
- Distributed network of servers taht deliver web content and services to users based on geo location
- Improve loading speeds with caching closer to end users
- Reduce latency and enhance UX
- Offer high availability and redundancy, minimizing risk of downtime
- POP locations = point of presence locations
- Increase reliability by spreading data to multiple locations

**Azure CDN**
- Managed service for CDN
- Global
- Scalable
- Secure

**How does it work?**
User requests a URL that targets edge server. If no cahced version exists, then CDN goes to origin and adds to cache for certain amount of time. Azure CDN default is 10 days, so users after the first get huge performance benefits.

Can set:
- Global caching rules: Set one rule for all endpoints in website
- Custom caching rules: Set rules per endpoint, get processed in order and override global when set

COntrolloing Cache Behavior:
- Bypass caching for query strings
- Override origin cache durations
- Set if missing: 

Can specifify cache expiration duration in days, hours, seconds
Matching conditions are **path** and **extension** (file extension)