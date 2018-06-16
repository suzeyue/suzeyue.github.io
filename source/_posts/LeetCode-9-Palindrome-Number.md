---
title: LeetCode-9.Palindrome Number
date: 2017-08-23 10:09:27
tags:
  - LeetCode
---

```
package leetcode

//Determine whether an integer is a palindrome. Do this without extra space.

func isPalindrome(x int) bool {
    if x < 0 || (x != 0 && x%10 == 0) {
        return false
    }
    
    tmp := 0
    for x > tmp {
        tmp = tmp * 10 + x % 10
        x /= 10
    }
    
    return (x == tmp || x == tmp / 10)
}
```
思路：
1. 整体循环除10来获取高位数
2. 低位循环乘10再相加获取低位的反转
3. 比较
