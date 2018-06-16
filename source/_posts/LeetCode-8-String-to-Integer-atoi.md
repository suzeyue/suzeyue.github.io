---
title: LeetCode-8.String to Integer (atoi)
date: 2017-08-22 16:16:30
tags:
  - LeetCode
---

```
package leetcode

//Implement atoi to convert a string to an integer.

func MyAtoi(str string) int {
    str = strings.TrimSpace(str)
    if len(str) < 1 {
        return 0
    }
    
    var neg int64 = 1
    if str[0] == '+' {
		str = str[1:]
	} else if str[0] == '-' {
		neg = -1
		str = str[1:]
	}
    
    var num int64
    
    for _, value := range str {
        if value < '0' || value > '9' {
            break
        }
        num = num * 10 + int64(value - '0') ;
        if num > math.MaxInt32 {
            break
        }
    }
    if num * neg >= math.MaxInt32 {
            return math.MaxInt32
    }
    if (num * neg <= math.MinInt32) {
        return math.MinInt32
    }
    
    return int(num * neg)
}
```
