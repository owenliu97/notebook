二分查找总有很多变种，在这道题中往往又容易找不到思路。

### 问题描述

在旋转一次的有序数组中找到目标值的位置。

旋转排序例如：`[0,1,2,4,5,6,7] might become [4,5,6,7,0,1,2]`

**Tags**: Array, Binary Search, Divide and Conquer

### 思路

首先，我们回顾一下二分查找的核心思路，其核心是 **每次查找都舍弃一半（数量级相当于一半）的元素**。两个要点：第一，保证每次迭代后目标范围缩小；第二，保证每次迭代都不会舍弃掉目标元素。

在本题中，确定舍弃的范围是解决的关键。给定左右指针和中间指针，本题需要先确认左右两半哪一半是有序的，然后再判断目标元素是否在有序的范围内。因此需要对左右和中点指针综合判断。

### 代码
```java
class Solution {
    public int search(int[] nums, int target) {
        if (nums == null || nums.length == 0) {
            return -1;
        }
        int l = 0;
        int r = nums.length - 1;
        while (l <= r) {
            int mid = (l + r) >>> 1;
            if (nums[mid] == target) {
                return mid;
            }
            if (nums[mid] >= nums[l]) {
                if (nums[mid] > target && nums[l] <= target) {
                    r = mid - 1;
                } else {
                    l = mid + 1;
                }
            } else {
                if (nums[mid] < target && nums[r] >= target) {
                    l = mid + 1;
                } else {
                    r = mid - 1;
                }
            }
        }
        return -1;
    }
}
```