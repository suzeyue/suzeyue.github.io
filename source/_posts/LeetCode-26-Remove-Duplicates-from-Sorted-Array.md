---
title: LeetCode-26.Remove Duplicates from Sorted Array
date: 2017-10-30 10:26:38
tags:
  - LeetCode
---
### Description
Given a sorted array, remove the duplicates in place such that each element appear only once and return the new length.
Do not allocate extra space for another array, you must do this in place with constant memory.
For example,
Given input array nums = `[1,1,2]` ,
Your function should return length = `2` , with the first two elements of nums being `1` and `2` respectively. It doesn't matter what you leave beyond the new length.

### code
```
func removeDuplicates(nums []int) int {
    length := len(nums)
    if length <= 1 {
        return length
    }
    
    result := 1
    for i := 1; i < length; i++ {
        if nums[i] != nums[i - 1] {
            nums[result] = nums[i]
            result++
        }
    }
    
    return result
}
```
