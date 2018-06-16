---
title: LeetCode-11.Container With Most Water
date: 2017-08-28 17:29:47
tags:
  - LeetCode
---

```
package leetcode

//Given n non-negative integers a1, a2, ..., an, where each represents a point at coordinate (i, ai). n vertical lines are drawn such that the two endpoints of line i is at (i, ai) and (i, 0).
//Find two lines, which together with x-axis forms a container, such that the container contains the most water.
//
//Note: You may not slant the container and n is at least 2.

import "math"

func MaxArea(height []int) int {
    left, right := 0, len(height) - 1
    maxArea := 0
    
    for ; left < right; {
        area := (right - left) * int(math.Min(float64(height[left]), float64(height[right])))
        maxArea = int(math.Max(float64(maxArea), float64(area)))
        if height[left] < height[right] {
            left++
        } else {
            right--
        }
    }
    
    return maxArea
}
```
