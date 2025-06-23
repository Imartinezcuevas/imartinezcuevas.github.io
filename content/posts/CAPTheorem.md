---
title: "CAP Theorem"
date: "2025-06-23"
summary: "A introduction to CAP theorem."
description: "CAP Theorem mini-essay."
toc: true
readTime: true
autonumber: true
math: false
tags: ["Python", "System Design"]
showTags: false
---

# CAP Theorem

## What is CAP Theorem?
You can only have 2 out of 3:
1. Consistency - all nodes/users see the same data at the same time
2. Availability - every request gets a response
3. Partition tolerance - system works despite network failures between nodes (**MUST**)

## What does this matter?
- Important to define the system characteristics early during non-functional requirements. The first thing we should do is start with CAP Theorem. We should ask ourselves: does this system needs to prioritize consistency or availability?
- They influence the design decisions during deep dives.

## How does it influence my desing?

If I choosed strong consistency:
- Implement distributed transactions
- Limit to a single node
- discuss consensus protocols
- accept higher latency

If I choose availabilit:
- Use multiple replicas
- CDC and eventual consistency is OK

### References
[CAP Theorem]([https://www.youtube.com/watch?v=VdrEq0cODu4)
