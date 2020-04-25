后端实习生第二轮面试中。由于自己之前做题的疏忽，没有总结复杂的二分查找算法，导致面试失利，特此总结！

### 问题描述

两个升序排序的数组，取平均数。要求时间复杂度 `O(log(m + n))`。

**Tags**: Array, Binary Search, Divide and Conquer

### 思路

首先，我们回顾一下二分查找的核心思路，其核心是 **每次查找都舍弃一半（数量级相当于一半）的元素**。两个要点：第一，保证每次迭代后目标范围缩小；第二，保证每次迭代都不会舍弃掉目标元素。

在本题中，两个数组同时进行迭代，为了简化指针的操作，我们一次只舍弃掉其中一个数组的一半元素，并对舍弃的元素进行计数，计数等于总数的一半时就得到了中位数。总长为偶数的，可以查找两次取平均值。本解法可以改变为找到两个排序数组的第 k 小的元素。

### 代码
```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int len = nums1.length + nums2.length;
        if (len % 2 == 0) {
            return (helper(nums1, 0, nums2, 0, len / 2) + helper(nums1, 0, nums2, 0, len / 2 + 1)) / 2.0;
        }
        return helper(nums1, 0, nums2, 0, len / 2 + 1);
    }
    private int helper(int[] nums1, int idx1, int[] nums2, int idx2, int k) {
        if (idx1 >= nums1.length) {
            return nums2[idx2 + k - 1];
        }
        if (idx2 >= nums2.length) {
            return nums1[idx1 + k - 1];
        }
        if (k == 1) {
            return Math.min(nums1[idx1], nums2[idx2]);
        }
        int p1 = idx1 + k / 2 - 1;
        int p2 = idx2 + k / 2 - 1;
        int mid1 = p1 < nums1.length ? nums1[p1] : Integer.MAX_VALUE;
        int mid2 = p2 < nums2.length ? nums2[p2] : Integer.MAX_VALUE;
        if (mid1 < mid2) {
            return helper(nums1, p1 + 1, nums2, idx2, k - k / 2);
        } else {
            return helper(nums1, idx1, nums2, p2 + 1, k - k / 2);
        }
    }
}
```