---
title: LeetCode-16.3Sum Closest
date: 2017-09-01 12:15:43
tags:
  - LeetCode
---

```
package leetcode

//Given an array S of n integers, find three integers in S such that the sum is closest to a given number, target.
//Return the sum of the three integers. You may assume that each input would have exactly one solution.
//
//For example, given array S = {-1 2 1 -4}, and target = 1.
//The sum that is closest to the target is 2. (-1 + 2 + 1 = 2).

import (
    "math"
    "sort"
)

func ThreeSumClosest(nums []int, target int) int {
	length := len(nums)
	if length < 3 {
		return math.MaxInt32
	}
	sort.Ints(nums)
	minClosest := math.MaxInt32

	for i := 0; i < length - 2; i++ {
		if i > 0 && nums[i] == nums[i-1] {
			continue
		}
		m,n := i + 1, length - 1
		for ; m < n; {
			closest := nums[i] +nums[m] + nums[n] - target
			if math.Abs(float64(minClosest)) > math.Abs(float64(closest)) {
				minClosest = closest
			}
			if closest == 0 {
				return target
			}
			if closest > 0 {
				n--
			} else {
				m++
			}

		}
	}

	return target + minClosest
}
```

3sum的变种，同样的思路。。。
