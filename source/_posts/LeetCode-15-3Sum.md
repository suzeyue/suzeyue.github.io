---
title: LeetCode-15.3Sum
date: 2017-08-31 14:15:46
tags:
  - LeetCode
---

```
package leetcode

import "sort"

//Given an array S of n integers, are there elements a, b, c in S such that a + b + c = 0?
//Find all unique triplets in the array which gives the sum of zero.

func ThreeSum(nums []int) [][]int {
	var result [][]int
	if len(nums) == 0 {
		return result
	}
	sort.Ints(nums)
	if nums[0] > 0 {
		return result
	}

	for i := 0; i < len(nums); i++ {
		if i > 0 && nums[i] == nums[i-1] {
			continue
		}
		target := 0 - nums[i]
		m, n := i+1, len(nums)-1
		for ; m < n; {
			if nums[m]+nums[n] == target {
				solution := []int{nums[i], nums[m], nums[n]}
				result = append(result, solution)
				for ; m < n && nums[m] == nums[m+1]; {
					m++
				}
				for ; m < n && nums[n] == nums[n-1]; {
					n--
				}
				m++
				n--
			} else if nums[m]+nums[n] < target {
				m++
			} else {
				n--
			}
		}
	}
	
	return result
}
```

3sum可以转化成之前的2sum问题：取第i个数，对大于i的数组进行2sum，target为 **0-第i个数** 。
注意：
1. 需要对数组排序
2. 去重
