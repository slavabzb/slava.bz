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

In order to solve the problem, we need to sum all the diagonal elements. We call an element *diagonal* if it's row index and column index are the same. Obviously, the algorithm has to take O(n) time to find the sum.

The first thing we should do is to understand how to process all the diagonal elements using only one loop iteration. For instance, let's consider a square matrix of size `2`.

![Square matrix of size 2](/assets/img/matrix-2x2.png)

As we can see, in that simple case all the elements are diagonal, e.g. the sum is (1 + 4) + (2 + 3) = 10. The indices (X, Y) of the diagonal 1-4 are changing together from 1 up to 2. For the other diagonal 2-3 row X-index is changing from 1 up to 23, whereas column Y-index is changing from 2 down to 1. Keeping this in mind, let's try to build the first version of the algorithm.

```python
def get_diagonal_sum(matrix):
    s = 0

    n = len(matrix)
    for i in range(n):
        s += matrix[i][i] + matrix[i][n - i - 1]

    return s

matrix = [
    [1, 2],
    [3, 4],
]

s = get_diagonal_sum(matrix)
print(s)
```

For our sample matrix this algorithm returns 10 and takes O(n) time. The key to this problem is to realize that this algorithm doesn't work for any matrices. Let's consider another example.

![Square matrix of size 3](/assets/img/matrix-3x3.png)

For that matrix the algorithm prints 30 instead of 25: central cell with value 5 is processed twice. Instead of making conditional check each loop iteration, we can subtract central cell before the result is returned.

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

As you can see, now the algorithm returns correct result for odd `N`. This solution takes O(n) time and works for any other edge cases, e.g. matrices of 1 element or empty matrices.
