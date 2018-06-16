---
title: LeetCode-27.Remove Element
date: 2017-10-30 10:46:21
tags:
  - LeetCode
---
### Description
Given an array and a value, remove all instances of that value in place and return the new length.
Do not allocate extra space for another array, you must do this in place with constant memory.
The order of elements can be changed. It doesn't matter what you leave beyond the new length.
Example:
Given input array nums = `[3,2,2,3]` , val = `3` 
Your function should return length = 2, with the first two elements of nums being 2.

### code
```
func removeElement(nums []int, val int) int {
    length := len(nums)
    if length < 1 {
        return 0
    }
    
    result := 0
    for i := 0; i < length; i++ {
        if nums[i] != val {
            nums[result] = nums[i]
            result++
        }
    }
    
    return result
}
```
