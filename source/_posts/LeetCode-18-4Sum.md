---
title: LeetCode-18.4Sum
date: 2017-09-06 11:37:39
tags:
  - LeetCode
---

```
package leetcode

import "sort"

//Given an array S of n integers, are there elements a, b, c, and d in S such that a + b + c + d = target? Find all unique quadruplets in the array which gives the sum of target.
//Note: The solution set must not contain duplicate quadruplets.
//
//For example, given array S = [1, 0, -1, 0, -2, 2], and target = 0.
//A solution set is:
//[
//  [-1,  0, 0, 1],
//  [-2, -1, 1, 2],
//  [-2,  0, 0, 2]
//]

func FourSum(nums []int, target int) [][]int {
    var res [][]int
    length := len(nums)
    if length < 4 {
        return res
    }
    sort.Ints(nums)
    
    for i := 0; i < length - 3; i++ {
        if i > 0 && nums[i] == nums[i - 1] {
            continue
        }
        
        for j := i + 1; j < length - 2; j++ {
            if j > i + 1 && nums[j] == nums[j - 1] {
                continue
            }
            
            left, right := j + 1, length - 1
            for ; left < right; {
                sum := nums[left] + nums[right] + nums[i] + nums[j]
                if sum == target {
                    solution := []int{nums[i], nums[j], nums[left], nums[right]}
                    res = append(res, solution)
                    for ; left < right && nums[left] == nums[left + 1]; {
                        left++
                    }
                    for ; left < right && nums[right] == nums[right - 1]; {
                        right--
                    }
                    left++
                    right--
                } else if sum < target {
                    left++
                } else {
                    right--
                }    

            }
        }
    }
    
    return res
}
```

4sum类似3sum，还是最终转换成2sum。
