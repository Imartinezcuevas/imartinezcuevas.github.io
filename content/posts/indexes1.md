---
title: "Hash indexes"
date: "2025-06-23"
summary: "A introduction to Hash indexes."
description: "Hash indexes mini-essay."
toc: true
readTime: true
autonumber: true
math: false
tags: ["Python", "System Design", "Data base"]
showTags: false
---

## Why use hash indexes?
The main objective of an index is to allow us to easily look up a key. With hash indexes, we literally use a hash map to store the keys and their corresponding values, where the values represent the location on disk. This allow for reads and writes in O(1) time.

However, there are some significant issues with hash maps:
- By design, hash maps aim to evenly distribute keys, which leads to poor performance on disk-based systems. This causes frequent disk seeks, resulting in time loss. To mitigate this, hash indexes are typically kept entirely in memory.
- RAM is expensive, and all keys must fit in it.
- Also, RAM is not durable, so we must use a Write-ahead log (WAL). It's literally a list with all writes and updates done to the database. This is going to slow down everything, since we need to write in the disk first.
- We can't do range queries. To perform one, we would have to iterate through the entire hash map, which is inefficient.

### References
[How do Hash Indexes work?]([https://www.youtube.com/watch?v=I1wQsY-Nh_k)
