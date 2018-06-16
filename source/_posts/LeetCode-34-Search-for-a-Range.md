---
title: LeetCode-34.Search for a Range
date: 2017-11-28 14:49:15
tags:
  - LeetCode
---
### Description
Given an array of integers sorted in ascending order, find the starting and ending position of a given target value.
Your algorithm's runtime complexity must be in the order of O(log n).
If the target is not found in the array, return `[-1, -1]` .

For example,
Given `[5, 7, 7, 8, 8, 10]` and target value 8,
return `[3, 4]` .

### Code
```
func searchRange(nums []int, target int) []int {
    lenN := len(nums)
    result := []int{-1, -1}
    if lenN < 1 {
        return result
    }

    head, tail := 0, lenN - 1
    for head < tail {
        mid := (head + tail) / 2
        if nums[mid] < target {
            head = mid + 1
        } else {
            tail = mid
        }
    }
    if nums[head] != target {
        return result
    }
    result[0] = head
    tail = lenN
    for head < tail {
        mid := (head + tail) / 2
        if nums[mid] <= target {
            head = mid + 1
        } else {
            tail = mid
        }
    }
    result[1] = tail - 1

    return result
}
```
