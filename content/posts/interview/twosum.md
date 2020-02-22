---
title: "The Twosum Problem in Linear Time and Constant Space"
date: 2020-02-21T16:09:52Z
---

I was recently introduced to the Two Sum problem during SINFO, a tech conference in Lisbon, hosted by IST (Instituto Superior Técnico).

During the conference, I saw a company offering 20€ to whomever solved their coding problem.
The problem is as follows:

---

We have an array of integers, for example:

```
[3, -6, 4, -3, 7, 4]
```

We want to find out whether there are two numbers that sum to zero:

- You need to do it in __*O(n)*__ time with constant additional space.
- You are allowed to modify the original array.

---

First things first, how does this relate to the two sum problem?

On [LeetCode](https://leetcode.com/problems/two-sum/), the problem is present as below:

> Given an array of integers, return indices of the two numbers such that they add up to a specific target.

> You may assume that each input would have exactly one solution, and you may not use the same element twice.

So in this case it is as if the target is always zero (and we don't care for indices). Knowing this we can now focus on solving the problem.

## Solution

Before I introduce the solution to the given problem, I will discuss other solutions that do not fit the constraints but are much more common.
In fact, I did a lot of Googling to find a solution (thinking I'm really smart, of course), and I did not find a single solution, I did not even find the problem...

### Naive

The naive solution is for each element, we compare it with every other element, sum the pair, check if it is zero.
The code looks like this:

```python
def naive(arr):
    for e1 in arr:
        for e2 in arr:
            if e1 + e2 == 0:
                return True
    return False
```

The solution is simple and readable, it has no dark magic and is also slooooooow.

This solution has a time complexity of __*O(n^2)*__, since for every element we need to iterate the whole array again.

We can make it a little better by not including elements we already checked, like so:

```python
def naive2(arr):
    for idx, e1 in enumerate(arr):
        for e2 in arr[idx + 1 :]:
            if e1 + e2 == 0:
                return True
    return False
```

However we still have __*O(n^2)*__, since for each iteration we do __n - current index__ it amounts to __n * ((n - 1) + (n - 2) + ... + (n - m))__ which comes out to be __n * n__.

### Hashing

For this one, we will need to allocate extra space! This means our space complexity is now up to at least __*O(n)*__.

We can use a set as our auxiliary data structure. 
It has **_O(1)_** insertion and membership check! 

*Note: this will only work for data structures relying on hashing*

```python
def hashing(arr):
    aset = set()
    for e in arr:
        if -e in aset:
            return True
        aset.add(e)
    return False
```

Since we just insert and check for elements inside our solution has **_O(n)_** time complexity.

### Sort

The following solutions are first and foremost dependent on the sorting algorithm used. 
If you use [Bogosort](https://en.wikipedia.org/wiki/Bogosort), your time complexity comes out to be **_O((n+1)!)_**, good luck!

In each I'll discuss the strategy after sorting.

#### Search

Using binary search we can look for the element in **_O(log(n))_** time, 
however we need to do it for every element, **n** times, (we could remove duplicates during sorting but shhhhh), 
this means our complexity comes out to be **_O(n log(n))_**.

```python
def sort_bin(arr):
    arr.sort()
    for e in arr:
        if binsearch(arr, -e):
            return True
    return False
```

#### Walk

This approach depends mostly on sorting, the search is merely comparing values from each of the array,
meaning we go through all the array, ending with **_O(n)_** time complexity.

```python
def sort_walk(arr):
    arr.sort()
    l = 0
    r = len(arr) - 1
    while l < r:
        s = arr[l] + arr[r]
        if s == 0:
            return True
        elif s < 0:
            l += 1
        else:
            r -= 1
    return False
```

### In-Place "Hashing"

This is the final solution to the problem, the one they actually wanted, and the only one that works.

The trick is to do an in-place, "hashing" data structure. 
Why the quotes? Because we can use modulus instead of fancy hashing, we are dealing with numbers after all.

For the sake of simplicity however, I will define an "hash" function.

```python
hash = lambda v : abs(v) % len(arr)
```

The `abs` call allows us to pretend the negative number is the positive one, this will make both end in the same index and allow for quick matching.

Before we start actually coding, we need to define the rules.

- If `hash(v)` equals the current index, the number is in the right place and we should not move it
- If `v` equals the symmetric of the value on index `hash(v)`, we found the pair and return
- If `hash(v)` equals `hash(arr[hash(v)])` we have a colision, we `continue` to the next iteration

After all of this, we didn't return or skip to the next iteration, we swap the current value with the value at `hash(v)`.

We do this until we find a pair.

Let's take a look at the code, shall we?

```python
def in_place_hashing(arr):
    hash = lambda v: abs(v) % len(arr)
    while True:
        for idx, value in enumerate(arr):
            c_idx = hash(value)         # The current value, real index
            t_idx = hash(arr[c_idx])    # The value at the destination, real index

            # If the current index is equal to the current hashing number there is nothing to do
            if idx == c_idx:
                continue

            # If the current value and the target sums to zero we found a pair
            if value + arr[c_idx] == 0:
                return True

            # If the target number is in the position it should be, theres nothing we can do
            if c_idx == t_idx and arr[t_idx] != 0:
                continue

            # Swap the numbers out
            v = arr[idx]
            arr[idx] = arr[c_idx]
            arr[c_idx] = v
    return False
```

Attentive readers will notice that the code has no stop condition in case the array does not contain any pair.

A simple fix is to count the number of changes in each iteration of the `while` loop, 
if the array did not suffer any change in the last `while` iteration we can stop.

The final code is as follows:

```python
def simple_zero_sum(arr):
    hash = lambda v: abs(v) % len(arr)
    changes = 1
    while changes > 0:
        changes = 0
        for idx, value in enumerate(arr):
            c_idx = hash(value)
            t_idx = hash(arr[c_idx])

            if idx == c_idx:
                continue

            if value + arr[c_idx] == 0:
                return True

            if c_idx == t_idx and arr[t_idx] != 0:
                continue

            v = arr[idx]
            arr[idx] = arr[c_idx]
            arr[c_idx] = v
            changes += 1
    return False
```

Voilá, O(1) space complexity, O(n) time complexity, O(0) real life usage.

I hope this has been useful for you!