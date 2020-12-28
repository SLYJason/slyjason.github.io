---
layout: post
title: "Segment Tree"
subtitle: "Range Query"
date: 2020-12-27
author: "Luyi"
header-img: "img/in-post/2020-12-27-segment-tree/post-bg.jpg"
tags: 
    - Data Structure
---

### What is Segment Tree?
From [Wikipedia](https://en.wikipedia.org/wiki/Segment_tree): In computer science, a segment tree, also known as a statistic tree, is a tree data structure used for storing information about intervals, or segments. In short, it is a data structure that allows answering range queries over an array effectively, while still being flexible enough to allow modifying the array.

### Motivation
Given an array like `[1,2,3,4,5,6]`, we want to support two of operations:
* `query(arr, i, j)`: Get range sum in `[i, j]`. For example, `query(arr, 1, 3)` should return `2 + 3 + 4 = 9`.
* `update(arr, i, val)`: Update value at index `i` with `val`. For example, after `update(arr, 1, 9)` then array becomes `[1,9,3,4,5,6]`.

There has 2 ways to solve this problem, let's examine one by one:

#### Option 1: Brute Force
* `query(arr, i, j)`: just get cumulative sum from `i` to `j`, so time complexity is `O(N)`.
* `update(arr, i, val)`: set index `i` value to `val` directly, so time complexity is `O(1)`.

#### Option 2: Prefix Sum
Making an additional array called `prefixSum` then we store prefix sum in the range `[0, i]`. For example, `[1,2,3,4,5,6]` prefix sum array is `[1,3,6,10,15,21]`.
* `query(arr, i, j)`: using `prefixSum` array we can easily get the range sum `[i, j]`, i.e. `query(arr, i, j) = prefixSum[j] - prefixSum[i-1]`, so time complexity is `O(1)`.
* `update(arr, i, val)`: since we are using `prefixSum`, so every update a value in the array need to update the whole `prefixSum` again, so time complexity is `O(N)`.

#### Comparison
From above analysis, we can easily get the **tradeoffs** of the above operations:

| Time Complexity | query | update |
| :-------------: | :---: | :----: |
| Brute Force     | O(N)  |  O(1)  |
| Prefix Sum      | O(1)  |  O(N)  |

Think about if the array is pretty large, and we have tremendous `query` and `update` operations, then above methods is very slow. So segment tree is used to solve these problems which both `query` and `update` will give us **logarithmic** time complexity.

### Definition
Given an array, we compute and store the sum of the elements of the whole array, i.e. the sum of the segment `arr[0...n−1]`. We then split the array into two halves `arr[0...n/2]` and `arr[n/2+1...n-1]` and compute the sum of each halve and store them. Each of these two halves in turn also split in half, their sums are computed and stored. This process repeats until all segments reach size 1. In other words we start with the segment `arr[0...n−1]`, split the current segment in half (if it has not yet become a segment containing a single element), and then calling the same procedure for both halves. For each such a segment we store the sum of the numbers on it.

We can say, that these segments form a binary tree: the root of this tree is the segment `arr[0...n−1]`, and each vertex (except leaf vertices) has exactly two children vertices. This is why the data structure is called **Segment Tree**, even though in most implementations the tree is not constructed explicitly.

For example, given an array `[1,2,3,4,5,6]`, its segment tree is looks like:

![Figure_1](/img/in-post/2020-12-27-segment-tree/Figure_1.svg).

As you can see from the above picture, each green node (leaf node) represents a single entry of the array. So we use this data structure to `query` and `update` in **logarithmic** time manner.

### Build Segment Tree
To build a segment tree, we will use an additional array to store segment sum. So question is how much space we needed? Here is the rule of thumb:
> A segment tree for an n element range can be comfortably represented using an array of size 4 * n.

To see why need such memory, you can go: [Stack Overflow](https://stackoverflow.com/questions/28470692/how-is-the-memory-of-the-array-of-segment-tree-2-2-ceillogn-1). 

Next step is we have to implement several methods of segment tree, namely: `build_tree`, `update_tree` and `sumRange`. Here is the full implementation:
``` java
class SegmentTree {
    int[] segment_tree;
    int[] arr;
    public SegmentTree(int[] arr) {
        segment_tree = new int[arr.length * 4];
        this.arr = arr;
        build_tree(0, 0, arr.length-1);
    }

    /**
     * Build segment tree.
     * @param index: segment tree index.
     * @param start: array start index.
     * @param end: array end index.
     */
    private void build_tree(int index, int start, int end) {
        if(start == end) {
            segment_tree[index] = arr[start];
            return;
        }
        int left_index = 2 * index + 1;
        int right_index = 2 * index + 2;

        int mid = start + (end - start)/2;
        build_tree(left_index, start, mid);
        build_tree(right_index, mid+1, end);

        segment_tree[index] = segment_tree[left_index] + segment_tree[right_index];
    }

    /**
     * Update segment tree.
     * @param index: segment tree index.
     * @param start: array start index.
     * @param end: array end index.
     * @param i: array update index.
     * @param val: array update value.
     */
    public void update_tree(int index, int start, int end, int i, int val) {
        if(start == end) {
            segment_tree[index] = val;
            arr[i] = val;
            return;
        }
        int left_index = 2 * index + 1;
        int right_index = 2 * index + 2;

        int mid = start + (end - start)/2;
        if(i <= mid) {
            update_tree(left_index, start, mid, i, val);
        } else {
            update_tree(right_index, mid+1, end, i, val);
        }

        segment_tree[index] = segment_tree[left_index] + segment_tree[right_index];
    }

    /**
     * Get array range sum [i, j].
     * @param index: segment tree index.
     * @param start: array start index.
     * @param end: array end index.
     * @param i: array range sum start index i.
     * @param j: array range sum end index j.
     * @return range sum [i, j].
     */
    public int sum_range(int index, int start, int end, int i, int j) {
        if(i > end || j < start) return 0; // [start, end] is out of the range [i, j].
        if(start >= i && end <= j) return segment_tree[index]; // [start, end] is in the range [i, j].

        int left_index = 2 * index + 1;
        int right_index = 2 * index + 2;

        int mid = start + (end - start)/2;
        int left_sum = sum_range(left_index, start, mid, i, j);
        int right_sum = sum_range(right_index, mid+1, end, i, j);
        return left_sum + right_sum;
    }
}
```

### Range Sum Query
Then back to our problem to use segment tree to `query` and `update` in **logarithmic** time manner, we define a class called `RangeSumQuery`:
```java
class RangeSumQuery {
    int[] arr;
    SegmentTree segmentTree;
    public RangeSumQuery(int[] arr) {
        this.arr = arr;
        segmentTree = new SegmentTree(arr);
    }

    public void update(int i, int val) {
        segmentTree.update_tree(0, 0, arr.length-1, i, val);
    }

    public int sumRange(int i, int j) {
        return segmentTree.sum_range(0, 0, arr.length-1, i, j);
    }
}
```
Now we can easily use above class to efficiently call `query` and `update`.

### Conclusion
In essence, segment tree uses **binary search** spirit to efficiently `query` and `update`, that's why we can have **logarithmic** time complexity of the two operations. In this article, we talked about the *Range Sum Query*, but segment tree can also be used in *Range Max/Min Query* and other advanced areas, you can explore it in the future.

### Reference
Here is several useful links:
* [Youtube: Segment Tree](https://www.youtube.com/watch?v=e_bK-dgPvfM&ab_channel=%E9%BB%84%E6%B5%A9%E6%9D%B0).
* [Segment Tree Theory and Applications](http://maratona.ic.unicamp.br/MaratonaVerao2016/material/segment_tree_lecture.pdf).
* [CP-Algorithms](https://cp-algorithms.com/data_structures/segment_tree.html#advanced-versions-of-segment-trees).
* [LeetCode: Segment Tree](https://leetcode.com/articles/a-recursive-approach-to-segment-trees-range-sum-queries-lazy-propagation/).
