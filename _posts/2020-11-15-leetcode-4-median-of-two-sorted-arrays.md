---
layout: post
title: "LeetCode 4: Median of Two Sorted Arrays"
subtitle: "Binary Search"
date: 2020-11-15
author: "Luyi"
header-img: "img/in-post/2020-11-15-leetcode-4-median-of-two-sorted-arrays/post-bg.jpg"
tags: 
    - LeetCode
---

This is a traditional [LeetCode](https://leetcode.com/problems/median-of-two-sorted-arrays/) challenge, but it has a few important things we have to dig it, so let's take a review how to solve it.

### Problem Statement
Link: [LeetCode](https://leetcode.com/problems/median-of-two-sorted-arrays/).
```
Given two sorted arrays nums1 and nums2 of size m and n respectively, return the median of the two sorted arrays.
```
Example:
```
Input: nums1 = [1,3], nums2 = [2].
Output: 2.00000.
Explanation: merged array = [1,2,3] and median is 2.
```
The statement is pretty clear, so let's figure out how to solve it!

### What is Median?
From [Wikipedia](https://en.wikipedia.org/wiki/Median): **a median is a value separating the higher half from the lower half of a data sample, a population or a probability distribution.** More clear is median is used for dividing a set into two equal length subnets, that one is always greater than the other.

![Figure_1](/img/in-post/2020-11-15-leetcode-4-median-of-two-sorted-arrays/Figure_1.svg)

### Solution 1: Two Pinter
An intuitive thought is to merge these two arrays and then calculate the median, most people can solve it fair easy:
```java
public double findMedianSortedArrays(int[] nums1, int[] nums2) {
    int[] merged = new int[nums1.length + nums2.length];
    int i=0, j=0, idx = 0;
    while(i < nums1.length && j < nums2.length) {
        merged[idx++] = nums1[i] < nums2[j] ? nums1[i++] : nums2[j++];
    }
    while(i < nums1.length) merged[idx++] = nums1[i++];
    while(j < nums2.length) merged[idx++] = nums2[j++];
    return merged.length % 2 == 0 ?
            (merged[merged.length/2-1] + merged[merged.length/2]) * 0.5 : (double)merged[merged.length/2];
}
```
It's a straightforward solution, so we will not discuss it here.

### Solution 2: Binary Search
#### Thoughts on the median
One of the rule of thumb is when we see some problems has sorted input, there must have a solution that is `O(logN)` time complexity, so it gives us a hint that we can solve it using **binary search**.

Let's think what is looks like when we get the median when we merge the two sorted array:
![Figure_2](/img/in-post/2020-11-15-leetcode-4-median-of-two-sorted-arrays/Figure_2.svg)

From above figure, we can see the merged array `Left Side` has 3 elements `1,3,5` from `A` and 2 elements: `2,4` from `B`. Similar, `Right Side` has 2 elements `7,9` from A and 3 elements `8,10,12` from `B`. 

This implies if we can figure out how to find an anchor to cut `A` then we know how many elements in left side of anchor will be placed in `Left Side` of merged array. Then we know how to place the anchor in `B` because we just need `N/2 - elements from left side in A anchor`. Here `N` means total elements from `A` and `B`. So we can use this anchor to calculate the median of the two sorted array.
 
#### Thoughts on the Anchor
![Figure_3](/img/in-post/2020-11-15-leetcode-4-median-of-two-sorted-arrays/Figure_3.svg)
From the above figure, in the array `A`, `L1` means the left element of the anchor and `R1` means the right element of the anchor. Similar, in the array `B`, `L2` means the left element of the anchor and `R2` means the right element of the anchor. These elements can be used to get the median, and they meet:
```
L1 < R1 AND L1 < R2
L2 < R2 AND L2 < R1
```
Now we can use `L1`, `L2`, `R1` and `R2` to get the median!
#### Coding
```java
public double findMedianSortedArrays(int[] nums1, int[] nums2) {
    if(nums1.length > nums2.length) return findMedianSortedArrays(nums2, nums1);
    int len = nums1.length + nums2.length;
    int cut1 = 0, cut2 = 0;
    int left = 0, right = nums1.length;
    while(cut1 <= nums1.length) {
        cut1 = left + (right - left)/2;
        cut2 = len/2 - cut1;
        double L1 = cut1 == 0 ? Integer.MIN_VALUE : nums1[cut1-1];
        double L2 = cut2 == 0 ? Integer.MIN_VALUE : nums2[cut2-1];
        double R1 = cut1 == nums1.length ? Integer.MAX_VALUE : nums1[cut1];
        double R2 = cut2 == nums2.length ? Integer.MAX_VALUE : nums2[cut2];
        if(L1 > R2) {
            right = cut1 - 1;
        } else if (L2 > R1) {
            left = cut1 + 1;
        } else {
            if(len % 2 == 0) {
                L1 = L1 > L2 ? L1 : L2;
                R1 = R1 > R2 ? R2 : R1;
                return (L1 + R1)/2;
            } else {
                return R1 > R2 ? R2 : R1;
            }
        }
    }
    return -1;
}
```
First of all, an important thing we need to note is we are only caring to cut the shortest length array, the another one anchor can be determined easily, so that why we `if(nums1.length > nums2.length) return findMedianSortedArrays(nums2, nums1);`

To known where is the anchor should be placed, we use `cut1` to denote how many elements on the left side of the anchor in `A`, `cut2` to denote how many elements on the left side of the anchor in `B`:
![Figure_4](/img/in-post/2020-11-15-leetcode-4-median-of-two-sorted-arrays/Figure_4.svg)
Then we can get the `L1`, `L2`, `R1`, `R2` easily using `cut1` and `cut2`. We apply binary search on `A` to get `cut1` and `cut2`, and then compare `L1` with `R2` and `L2` with `R1` respectively:
```
If L1 > R2: means cut1 has more elements, we need to decrease cut1, in the mean time, we increase cut2.
If L2 > R1: means cut2 has more elements, we need to decrease cut2, in the mean time, we increase cut1.
```
Finally, if the total length of `A` and `B` is even, we get median using `(max(L1, L2) + min(R1, R2)) / 2`. If the length is odd, we get median using `min(R1, R2)`.

An important thing need to note is `cut1` and `cut2` may reach out the boundary, so we need to assign `L1`, `L2`, `R1`, `R2` as `MIN_VALUE` and `MAX_VALUE` respectively. Time complexity of this implementation is `O(log(min(M, N)))`, because we only cut the minimum length array, space complexity is `O(1)`.

### Follow Up
What if we want the median of K sorted arrays? We can treat it as find **Kth minimum element in sorted arrays**, the logic is:
* Create a `min-heap`(PriorityQueue), push all the 0-th element into the queue.
* Poll the min from the queue and add another element into the queue where the min belongs to that array.
* Repeat it until reach the Kth element, where Kth is median of these sorted arrays.