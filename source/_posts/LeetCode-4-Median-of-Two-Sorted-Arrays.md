---
title: LeetCode-4.Median of Two Sorted Arrays
date: 2017-08-18 16:09:21
tags:
  - LeetCode
---

```
package leetcode

//There are two sorted arrays nums1 and nums2 of size m and n respectively.
//Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).
//
//Example 1:
//nums1 = [1, 3]
//nums2 = [2]
//
//The median is 2.0
//
//Example 2:
//nums1 = [1, 2]
//nums2 = [3, 4]
//
//The median is (2 + 3)/2 = 2.5

func FindMedianSortedArrays(nums1 []int, nums2 []int) float64 {
    lenTotal := (len(nums1) + len(nums2))
    
    if i := lenTotal % 2; i == 1 {
        return getKth(nums1, nums2, lenTotal / 2 + 1)
    } else {
        return (getKth(nums1, nums2, lenTotal / 2) + getKth(nums1, nums2, lenTotal / 2 + 1)) / 2
    }
}

func getKth(nums1, nums2 []int, k int) float64 {
    len1, len2 := len(nums1), len(nums2)
    if len1 > len2 {
        return getKth(nums2, nums1, k)
    }
    if len1 == 0 {
        return float64(nums2[k - 1])
    }
    if k == 1 {
        return float64(min(nums1[0], nums2[0]))
    }
    
    i := min(len1, k / 2)
    j := min(len2, k / 2)
    if nums1[i - 1] > nums2[j - 1] {
        return getKth(nums1, nums2[j:], k - j)
    } else {
        return getKth(nums1[i:], nums2, k - i)
    }
}

func min(x, y int) int {
    if x < y {
        return x
    } else {
        return y
    }
}
```
