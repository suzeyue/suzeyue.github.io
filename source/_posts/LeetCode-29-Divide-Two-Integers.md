---
title: LeetCode-29.Divide Two Integers
date: 2017-11-01 15:04:54
tags:
  - LeetCode
---
### Description
Divide two integers without using multiplication, division and mod operator.
If it is overflow, return MAX_INT.

实现除法，但不允许使用乘、除、取余操作。第一反应就是使用位移。
主要就是循环把除数左移一位，如果小于被除数，那么结果就加上 `1` 左移几位的结果，然后被除数减去位移后的除数。

### Code
```
func divide(dividend int, divisor int) int {
    var m int64 = int64(abs(dividend))
    var n int64 = int64(abs(divisor))
    var res int64 = 0
    for m >= n {
        t := n
        var p int64 = 1
        for m > (t << 1) {
            t <<= 1
            p <<= 1
        }
        res += p
        m -= t
    }
    if dividend < 0 && divisor > 0 || dividend > 0 && divisor < 0 {
        res = -res
    }
    if res > math.MaxInt32 {
        return math.MaxInt32
    } else {
        return int(res)
    }
}

func abs(x int) int {
    if x < 0 {
        return -x
    }
    return x
}
```
