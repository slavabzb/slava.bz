---
layout: post
title:  "Are you smart enough to work in Yandex?"
date:   2017-12-14 12:20:45 +0500
categories: python algorithms
---
# Are you smart enough to work in Yandex?

## Sum of diagonals

Find the sum of the diagonals of the squared matrix.

---

Let's consider a squared matrix M.

```python
def get_diagonal_sum(matrix):
    s = 0

    n = len(matrix)
    for i in range(n):
        s += matrix[i][i] + matrix[i][n - i - 1]

    if n % 2:
        i = int((n - 1) / 2)
        return s - matrix[i][i]

    return s
```
