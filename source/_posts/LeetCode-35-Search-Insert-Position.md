---
title: LeetCode-35.Search Insert Position
date: 2017-12-27 22:00:57
tags:
  - LeetCode
---
### Description
Given a sorted array and a target value, return the index if the target is found. If not, return the index where it would be if it were inserted in order.
You may assume no duplicates in the array.

```
Example 1:
Input: [1,3,5,6], 5
Output: 2
```

```
Example 2:
Input: [1,3,5,6], 2
Output: 1
```

```
Example 3:
Input: [1,3,5,6], 7
Output: 4
```

```
Example 4:
Input: [1,3,5,6], 0
Output: 0
```

### Code

```
func searchInsert(nums []int, target int) int {
    if len(nums) <= 0 {
        return -1
    }
    
    left, right := 0, len(nums) - 1
    for left <= right {
        mid := (left + right) / 2
        if nums[mid] == target {
            return mid
        }
        if nums[mid] > target {
            right = mid - 1
        } else {
            left = mid + 1
        }
    }
    
    return left
}
```

上面的代码 `mid := (left + right) / 2` ，如果 `left` 和 `right` 很大的话，会出现溢出错误，应改为 `mid := left + (right - left) / 2` 。前面也出现了类似的问题，现在才发现。。。
