---
title: LeetCode-33.Search in Rotated Sorted Array
date: 2017-11-17 11:17:40
tags:
  - LeetCode
---
### Description
Suppose an array sorted in ascending order is rotated at some pivot unknown to you beforehand.
(i.e., `0 1 2 4 5 6 7` might become `4 5 6 7 0 1 2` ).
You are given a target value to search. If found in the array return its index, otherwise return -1.
You may assume no duplicate exists in the array.

参考：[Grandyang的博客](http://www.cnblogs.com/grandyang/p/4325648.html)

### Code
```
func search(nums []int, target int) int {
    if len(nums) == 0 {
        return -1
    }
    
    left, right := 0, len(nums) - 1
    for left <= right {
        mid := (left + right) / 2
        if nums[mid] == target {
            return mid
        } else if nums[mid] < nums[right] {
            if nums[mid] < target && nums[right] >= target {
                left = mid + 1
            } else {
                right = mid - 1
            }
        } else {
            if nums[left] <= target && nums[mid] > target {
                right = mid - 1
            } else {
                left = mid + 1
            }
        }        
    }
    
    return -1
}
```
