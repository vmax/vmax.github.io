---
title: Python concurrency gotcha
date: 2021-01-15
description: and how to avoid it
---

Asynchronous functions are usually a good way to optimize I/O bound tasks. However,
one must be careful passing values to different threads, especially those that change.
What do you think this code would output?

```python
from concurrent.futures import ThreadPoolExecutor
ex = ThreadPoolExecutor()
for i in range(100):
    ex.submit(lambda: print(i, end=', '))
```

Well, before I'd answer it'd print all 100 numbers but not in correct order. However, this is
completely wrong. This is what I got running on my machine (you result would vary):

```
0, 2, 2, 5, 5, 5, 9, 9, 9, 9, 14, 14, 14, 14, 14, 20, 20, 20, 20, 20, 20, 27, 27, 27, 27, 27, 27, 27, 35, 35, 35, 35, 35, 35, 35, 35, 44, 44, 44, 44, 44, 44, 44, 44, 44, 54, 54, 54, 54, 54, 54, 54, 54, 54, 54, 65, 65, 65, 65, 65, 65, 65, 65, 65, 65, 65, 77, 77, 77, 77, 77, 77, 77, 77, 77, 77, 77, 77, 99, 99, 99, 99, 99, 99, 99, 99, 99, 99, 99, 99, 99, 99, 99, 99, 99, 99, 99, 99, 99, 99,
```

The reason for it is that `i`, being shared between all threads (main and ones in executor), is getting modified *before* it gets a chance to be printed.

A way to overcome it would be to make sure that we don't use the variable itself but its value **at the time we submit a task to the executor**.
Simplest way to do that would be to use [`functools.partial`](https://docs.python.org/3/library/functools.html#functools.partial):

>The partial() is used for partial function application which “freezes” some portion of a function’s arguments and/or keywords resulting in a new object with a simplified signature.

The *"freezes"* bit is the one we're interested in.

Code sample:


```python
from concurrent.futures import ThreadPoolExecutor
from functools import partial

ex = ThreadPoolExecutor()

for i in range(100):
    ex.submit(partial(print, i, end=', '))
```

As expected, this one would print all numbers:

```
0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99,
```

With operations more complex than print, the order would vary based on which thread completes the task earlier.
