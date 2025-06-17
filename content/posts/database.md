---
title: "Database indexes"
date: "2025-06-16"
summary: "A introduction to databases indexes."
description: "Database indexes mini-essay."
toc: true
readTime: true
autonumber: true
math: false
tags: ["Python"]
showTags: false
---

## Why are database indexes important?

Indexes in databases are auxiliary structures that help us access information more efficiently and quickly. Their main function is to speed up read operations, specially those that search for specific rows or filter by certain columnss. Without indexes, a database would have to scan the entire table to find the requested data, which would be very costly in terms of time and resources. In such cases, read and write operations would have a time complexity of $O(n)$.

One way to optimize writes is to always append data at the end of the table, witch can result in $O(1)$ write complexity. However, this would make reads slower. Since fast reads are usually more desirable than fast writes in most applications, we need to use another techniques, like indexing, to solve this trade-off.

That said, indexes are not free. They consume aditional storage on disc and must be updated whenever the data change. For this reason, it's important to find the right balance: too many indixes can slow down write operations, while too few could lead to inefficient reads.

In summary, indexes are crucial to improve database performance. Designing them properly, deciding which column to index and when to do so, is a key part of the optimizing any system that handles large volumes of data.

### References
- [What is a Database Index?]([https://www.codecademy.com/article/sql-indexes))
- [Database Indexes: What do they do?]([https://www.youtube.com/watch?v=LzNdvuj3a5M&list=PLjTveVh7FakLdTmm42TMxbN8PvVn5g4KJ&index=2))
