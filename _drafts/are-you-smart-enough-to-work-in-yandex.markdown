---
layout: post
title:  "Are you smart enough to work in Yandex?"
date:   2017-12-14 12:20:45 +0500
categories: python algorithms
---
# Are you smart enough to work in Yandex?

List of problems

* [Set difference](#set-difference)
* [FIFO queue](#fifo-queue)
* [Palindrome](#palindrome)
* [Reverse linked list](#reverse-linked-list)
* [Sum of diagonals](#sum-of-diagonals)

## Set difference

`A` and `B` are sorted lists. Find elements contained in `A` and not contained in `B`.

---

The problem is to find a set difference between `A` and `B`, but we have lists instead of sets. The lists are sorted, we can use that to find the result effectively.

```python
def set_diff(a, b):
    i = 0
    j = 0
    result = []
    while i < len(a) and j < len(b):
        if a[i] < b[j]:
            result.append(a[i])
            i += 1
        elif a[i] == b[j]:
            i += 1
        else:
            j += 1
    result.extend(a[i:])
    return result
```

We start processing `A` and `B` from the beginnig. We compare current elements. If we find that an element from `A` is less, we will include it to the answer. Otherwise we move current positions `i` and `j` forward.

The loop terminates if all the elements from `A` or `B` were processed. If `B` was shorter, we have to add to the answer the rest elements from `A` starting from the current position.

This algorithm takes O(n+k) time and uses no additional memory.

## FIFO queue

Create a FIFO queue using two stacks.

---

FIFO queue is a queue where the oldest (first) item is processed first. In Python we have no special class for stacks, we can use lists instead.

The idea is to put incoming elements in the first stack. When the element is aquired, we have to turn the first stack and return an item from top of the second stack.

We can create a simple class that implements queue interface.

```python
class FifoQueue:
    def __init__(self):
        self._instack = []
        self._outstack = []

    def enqueue(self, item):
        self._instack.append(item)

    def dequeue(self):
        if not self._outstack:
            self._turn()
        return self._outstack.pop()

    def _turn(self):
        for _ in range(len(self._instack)):
            self._outstack.append(self._instack.pop())
```

The usage of the `FifoQueue` class is simple. We have only 2 public methods. When we need to store data, the method `enqueue` is used. When we need stored data back, we call the `dequeue` method.

## Palindrome

Check wheather a string is a palindrome or not. Non-alphanumeric characters are ignored. Case insensitive.

---

A string is called *a palindrome* if the string read from left to right is equal to the string read from right to left. For example, ignoring the difference between uppercase and lowercase letters and skipping non-alphanumeric characters, the strign `Abcba!` is a palindrome, while the string `Hello` is not.

Whereas there is a beautiful recursive solution, it takes additional memory per each recursion call and is not so effective as an iteration-based one.

```python
def is_palindrome(s):
    i = 0
    j = (len(s) or 1) - 1
    while i != j:
        if not s[i].isalnum():
            i += 1
            continue
        if not s[j].isalnum():
            j -= 1
            continue
        if s[i].lower() != s[j].lower():
            return False
        i += 1
        j -= 1
    return True
```

This algorithm takes O(n) time and no additional memory.

## Reverse linked list

Reverse given linked list.

---

For that problem we need to define a new data type for linked list. For example, it may looks like the following.

```python
from typing import Any, NoReturn, Optional


class Node:
    def __init__(self, value: Any, next: Optional['Node']) -> NoReturn:
        self.value = value
        self.next = next
```

Class `Node` defines a type for an item of the linked list. Each node has a payload (value of any type) and a pointer to the next node (the `next` attribute). If the node has no next node, we call it a *tail* node.

Using this data structure, a regular python list `[1, 2, 3, 4, 5]` can be represented like that.

```python
tail = Node(5, None)
z = Node(4, tail)
y = Node(3, z)
x = Node(2, y)
head = Node(1, x)
```

The problem is to reverse given linked list, i.e. make tail node a head for the new linked list.

Usually we process such structures using iteration. For example, a function that prints linked list can be written like that.

```python
def print_list(head: Node):
    current = head
    while current:
        print(current.value)
        current = current.next
```

Using such iteration pattern, we can walk through the list and swap the pointers of the two neighbour nodes.

```python
def reverse(head: Node) -> Node:
    previous = None
    current = head
    while current:
        tmp = current.next
        current.next = previous
        previous = current
        current = tmp
    return previous
```

This algorithm makes changes *in-place*, i.e. it modifies an existing list. We can accept such behaviour for the sake of taking O(n) time and requiring no additional memory.

## Sum of diagonals

Find the sum of the diagonals of the square matrix.

---

In order to solve the problem, we need to sum all the diagonal elements. We call an element *diagonal* if it's row index and column index are the same. Obviously, the algorithm has to take O(n) time to find the sum.

The first thing we should do is to understand how to process all the diagonal elements using only one loop iteration. For instance, let's consider a square matrix of size `4`.

<p align="center">
<img src="/assets/img/matrix-4x4.png" alt="Square matrix of size 4"  width="400px"/>
</p>

As we can see, in that case the sum is (1 + 6 + 11 + 16) + (4 + 7 + 10 + 13) = 68. The indices (X, Y) of the diagonal 1-6-11-16 are changing together from 1 up to 4. For the other diagonal 4-7-10-13 row X-index is changing from 1 up to 4, whereas column Y-index is changing from 4 down to 1. Keeping this in mind, let's try to build the first version of the algorithm.

```python
def get_diagonal_sum(matrix):
    s = 0

    n = len(matrix)
    for i in range(n):
        s += matrix[i][i] + matrix[i][n - i - 1]

    return s

matrix = [
    [1, 2, 3, 4],
    [5, 6, 7, 8],
    [9, 10, 11, 12],
    [13, 14, 15, 16],
]
s = get_diagonal_sum(matrix)
print(s)
```

For our sample matrix this algorithm returns 68 and takes O(n) time. Let's consider another example.

<p align="center">
<img src="/assets/img/matrix-3x3.png" alt="Square matrix of size 3"  width="400px"/>
</p>

For that matrix the algorithm returns 30 instead of 25: central cell with value 5 is processed twice. Instead of making conditional check each loop iteration, we can subtract central cell before the result is returned.

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

As you can see, now the algorithm returns correct result. This solution takes O(n) time and works for any other edge cases, e.g. matrices of 1 element or empty matrices.
