---
title: LeetCode-1.Two Sum
date: 2017-08-15 13:45:27
tags:
  - LeetCode
---

```
package leetcode

//Given an array of integers, return indices of the two numbers such that they add up to a specific target.
//You may assume that each input would have exactly one solution, and you may not use the same element twice.
//
//Example:
//Given nums = [2, 7, 11, 15], target = 9,
//Because nums[0] + nums[1] = 2 + 7 = 9,
//return [0, 1].
func TwoSum(nums []int, target int) []int {

    targetMap := make(map[int]int)
    result := make([]int, 2)

    for i, v := range nums {
        if index, ok := targetMap[target - v]; ok {
            result[0] = index
            result[1] = i
            break
        } else {
            targetMap[v] = i
        }
    }

    return result
}
```
