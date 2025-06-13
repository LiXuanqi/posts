---
title: 4 Basic Sort Algorithms 
date: "2018-11-25T22:12:03.284Z"
description: "Hello World"
tags: ['Algorithm']
visible: true
---

## Overview
The article introduces 4 sort algorithms.
- Selection sort
- Quicksort
- Merge sort
- Bucket sort

You can practice [here](https://www.lintcode.com/problem/sort-integers/description)
## Selection sort
![image](https://upload.wikimedia.org/wikipedia/commons/thumb/b/b0/Selection_sort_animation.gif/250px-Selection_sort_animation.gif)
We define the list as two parts:
- the sublist of items already sorted.
- the sublist of items remaining to be sorted.
At the beginning,  the sorted sublist is empty and the unsorted sublist is the entire input list. The algorithm proceeds by finding the smallest (or largest, depending on sorting order) element in the unsorted sublist, exchanging (swapping) it with the leftmost unsorted element (putting it in sorted order), and moving the sublist boundaries one element to the right.
```java
public class Solution {
    /**
     * @param A: an integer array
     * @return: nothing
     */
    public void sortIntegers(int[] A) {
        // write your code here
        if (A == null || A.length == 0) {
            return;
        }
        // make one element at right position in one round.
        for (int i = 0; i < A.length; i++) {
            int index = findSmallest(A, i);
            swap(A, index, i);
        }
    }
    private void swap(int[] A, int left, int right) {
        int temp = A[left];
        A[left] = A[right];
        A[right] = temp;
    }
    private int findSmallest(int[] A, int startIndex) {
        int ans = startIndex;
        for (int i = startIndex; i < A.length; i++) {
            if (A[ans] > A[i]) {
                ans = i;
            }
        }
        return ans;
    }
}
```
- Time complexity: $O(n^2)$
- Space complexity: O(1)
- unstable
## Merge sort
![image](https://upload.wikimedia.org/wikipedia/commons/c/cc/Merge-sort-example-300px.gif)

The core idea is divide and conquer.

We divide list to two parts. If we can sort them separately in some way, the problem changes to merge two sorted list, we can use two pointers to do it. The base case is there are only one element in array. we do not need to sort it and just return itself.
```java
public class Solution {
    /**
     * @param A: an integer array
     * @return: nothing
     */
    public void sortIntegers(int[] A) {
        // write your code here
        if (A == null || A.length == 0) {
            return;
        }
        mergeSort(A, 0, A.length - 1, new int[A.length]);
    }
    private void mergeSort(int[] nums, int start, int end, int[] temp) {
        if (start >= end) {
            return;
        }
        int mid = start + (end - start) / 2;
        mergeSort(nums, start, mid, temp);
        mergeSort(nums, mid + 1, end, temp);
        merge(nums, start, end, temp);
    }
    private void merge(int[] nums, int start, int end, int[] temp) {
        int mid = start + (end - start) / 2;
        int index = start;
        int pointer1 = start;
        int pointer2 = mid + 1;
        while (pointer1 <= mid && pointer2 <= end) {
            if (nums[pointer1] < nums[pointer2]) {
                temp[index++] = nums[pointer1++];
            } else {
                temp[index++] = nums[pointer2++];
            }
        }
        while (pointer1 <= mid) {
            temp[index++] = nums[pointer1++];
        }
        while (pointer2 <= end) {
            temp[index++] = nums[pointer2++];
        }
        // copy from temp to original array.
        for (int i = start; i <= end; i++) {
            nums[i] = temp[i];
        }
    }
}
```
Question: 
- why we need `int[] temp`?
because we can't finish `merge()` in-place, we should also notice where to new this temp array. If we new it in `mergeSort()`, it will be called many times in the process of recursion.

### Algorithm Complexity

#### Time complexity: 
O(nlogn + n ) => O(nlogn)

It divide to two parts: divide and merge.

- For divide: O(1 + 2 + 4 + ... + n/2) = O(n)
   - For first level, we need cut 1 time.
   - For second level, we neeed cut 2 times.
   - For n level, there are n nodes, so we need cut n/2 times.
- For merge: O(nlogn)
   - For every level, there are always n elements remaining to be merged.

#### Space complexity: 

O(n + logn) => O(n)

1. `int[] temp`: O(n)
2. call stack: O(the height of recursion tree) = O(logn)

- stable
## Quicksort
![image](https://upload.wikimedia.org/wikipedia/commons/6/6a/Sorting_quicksort_anim.gif)
There are 2 steps:
1. choose a pivot and divide the list to two parts, all elements on the left side of pivot are all <= pivot, all elements on the right side of pivot are all >= pivot.
2. do quickSort for the two parts recursively. 
```java
public class Solution {
    /**
     * @param A: an integer array
     * @return: nothing
     */
    public void sortIntegers(int[] A) {
        // write your code here
        if (A == null || A.length == 0) {
            return;
        }
        quickSort(A, 0, A.length - 1);
    }
    private void quickSort(int[] nums, int start, int end) {
        if (start >= end) {
            return;
        }
        int mid = start + (end - start) / 2;
        int pivot = nums[mid];
        int left = start;
        int right = end;
        // why left <= right not left < right?
        while (left <= right) {
            // why nums[left] < pivot not nums[left] <= pivot?
            while (left <= right && nums[left] < pivot) {
                left++;
            }
            while (left <= right && nums[right] > pivot) {
                right--;
            }
            if (left <= right) {
                swap(nums, left, right);
                left++;
                right--;
            }
        }
        quickSort(nums, start, right);
        quickSort(nums, left, end);
    }
    private void swap(int[] nums, int left, int right) {
        int temp = nums[left];
        nums[left] = nums[right];
        nums[right] = temp;
    }
}
```
Questions:

- why left <= right not left < right?

If `left < right`, at the end of while loop, the left will equal to right, and then we will do quicksort for two intervals `[start, right]` and `[left, end]`. There is an overlap between two intervals. Take [1, 2] as an example, at the end of the code, we should do `quickSort` for [1, 2] again, which means we do not reduce the size of the problem and causes infinite recursion.

- why nums[left] < pivot not nums[left] <= pivot?

Assume the list = [1 1 1 1 1 1], if `nums[left] <= pivot`, the left pointer will move to the end of the list. 
```
[1 1 1 1 1 1]
         r l
```
and we will do `quickSort` for [start, right].
Also, we do not reduce the size of the problem.

### Algorithm Complexity
- Worst-case time complexity: O(n^2)
- average time complexity: O(nlogn)
- space complexity: O(n), worst case's the height of recursion tree is n.
- unstable

## bucket sort
[Leetcode 347. Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)
```java
class Solution {
    public List<Integer> topKFrequent(int[] nums, int k) {
        List<Integer> ans = new ArrayList<>();
        if (nums == null || nums.length == 0 || k <= 0) {
            return ans;
        }
        Map<Integer, Integer> map = new HashMap<>();
        for (int num : nums) {
            map.put(num, map.getOrDefault(num, 0) + 1);
        }
        List[] bukkits = new List[nums.length + 1];
        for (int key : map.keySet()) {
            int freq = map.get(key);
            if (bukkits[freq] == null) {
                bukkits[freq] = new ArrayList<>();
            }
            bukkits[freq].add(key);
        }
        // iterate from tail.
        for (int i = bukkits.length - 1; i >= 0; i--) {
            if (k <= 0) {
                break;
            }
            if (bukkits[i] != null) {
                k -= bukkits[i].size();
                ans.addAll(bukkits[i]);
            }
        }
        return ans;
    }
}
```
