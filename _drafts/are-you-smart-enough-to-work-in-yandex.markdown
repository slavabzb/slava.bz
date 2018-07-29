---
layout: post
title:  "Are you smart enough to work in Yandex?"
date:   2017-12-14 12:20:45 +0500
categories: python algorithms
---
# Are you smart enough to work in Yandex?

List of problems

[Sum of diagonals](#sum-of-diagonals)

## Sum of diagonals

Find the sum of the diagonals of the square matrix.

---

Let's consider a square matrix of size `N`. In order to solve the problem, we need to sum all the diagonal elements. We call an element *diagonal* if it's row index and column index are the same.

![alt text][/assets/img/matrix.png]

Obviously, our algorithm has to take O(n) time to find the sum.

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
