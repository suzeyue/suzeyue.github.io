---
title: LeetCode-31.Next Permutation
date: 2017-11-04 18:04:46
tags:
  - LeetCode
---
### Description
Implement next permutation, which rearranges numbers into the lexicographically next greater permutation of numbers.
If such arrangement is not possible, it must rearrange it as the lowest possible order (ie, sorted in ascending order).
The replacement must be in-place, do not allocate extra memory.
Here are some examples. Inputs are in the left-hand column and its corresponding outputs are in the right-hand column.
`1,2,3` → `1,3,2`
`3,2,1` → `1,2,3`
`1,1,5` → `1,5,1`


这道题是让求一组数的全排列的的下一项。
思路：对于数组 `nums` ,逆序遍历数组，当 `nums[i] < nums[i+1]` 时，将 `nums[i]` 与 `i` 之后大于 `nums[i]` 中最小的值与 `nums[i]` 互换，然后将 `i` 之后的部分逆序；如果遍历数组没有找到 `nums[i] < nums[i+1]` ，这就代表这是全排列的最后一项，直接将整个 `nums` 逆序就可以全排列的第一项。


### Code
```
func nextPermutation(nums []int)  {
	length := len(nums)
	for i := length -2; i >= 0; i-- {
		if nums[i + 1] > nums[i] {
			j := length - 1
			for ;j >= i; j-- {
				if nums[j] > nums[i] {
					break
				}
			}
			nums[i], nums[j] = nums[j], nums[i]
			reverse(nums[i + 1:])
			return
		}

	}
    reverse(nums)
}

func reverse(nums []int) {
    i, j := 0, len(nums) - 1
    for i < j {
        nums[i], nums[j] = nums[j], nums[i]
        i++
        j--
    }
}
```
