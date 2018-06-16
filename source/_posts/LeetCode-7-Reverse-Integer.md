---
title: LeetCode-7.Reverse Integer
date: 2017-08-21 11:43:16
tags:
  - LeetCode
---


```
package leetcode

//Reverse digits of an integer.
//
//Example1: x = 123, return 321
//Example2: x = -123, return -321

func Reverse(x int) int {
    res := 0
    for x != 0 {
        if abs(res) > math.MaxInt32 / 10 {
            return 0
        }
        res = res * 10 +  x % 10
        x /= 10
    }
    
    return res
}

func abs(x int) int {
    if x < 0 {
        return -x
    }
    if x == 0 {
  		return 0 // return correctly abs(-0)
  	}
    return x
}
```

主要注意的就是反转之后的溢出。
