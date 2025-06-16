---
title: "Context Managers"
date: "2025-06-16"
toc: true
readTime: true
autonumber: true
math: false
tags: ["Python"]
showTags: false
---

In Python, managing resources such as files, sockets or database connections is a common task. However, ensuring these resources are correctly acquired and released can quickly become error-prone. Context managers provide a structured and elegant solution to this problem.

At its core, a context manager is a class that facilitates the setup and teardown of resources, abstracting the cleanup logic.

Imagine opening a file without a context manager:
```python
f = open('data.txt')
data = f.read()
f.close()
```

If an error occurs during `f.read()`, `f.close()` is never called, leading to a file corruption or resource leaks. With a context manager:
```python
with open("data.txt") as f:
  data = f.read()
```
Now, no matter what happens inside the block, the file will be closed appropriately when exiting the context. This makes the code cleaner, robust and safer.

## The protocol behind the magic
Context managers rely on two special methods: `__enter__()` and `__exit__()`.
- `__enter__()` is executed when the block is entered. It typically acquires the resource and returns it.
- `__exit__()` is executed when exiting the block, even if due to an exception. It handles cleanup and can optionally suppress exceptions.

```python
with SomeContextManager() as resource:
  use(resource)
```

This is syntactic sugar for:
```python
ctx = SomeContextManager()
resource = ctx.__enter__()
try:
  use(resource)
finally:
  ctx.__exit__(*sys.exc_info())
```

### References
- [Context Managers and Python's with Statement]([https://dataintensive.net/](https://realpython.com/python-with-statement/))
- [Gestores de contexto]([https://www.databass.dev/](https://ellibrodepython.com/context-managers-python))
- [27. Context Managers]([https://api.drum.lib.umd.edu/server/api/core/bitstreams/17176ef8-8330-4a6c-8b75-4cd18c570bec/content](https://book.pythontips.com/en/latest/context_managers.html))
