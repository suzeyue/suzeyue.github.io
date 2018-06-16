---
title: LeetCode-13.Roman to Integer
date: 2017-08-30 10:59:48
tags:
  - LeetCode
---

```
package leetcode

//Given a roman numeral, convert it to an integer.
//Input is guaranteed to be within the range from 1 to 3999.

var value = map[string]int{
    "M": 1000,
    "D": 500,
    "C": 100,
    "L": 50,
    "X": 10,
    "V": 5,
    "I": 1,        
}


func RomanToInt(s string) int {
    result := 0
    for i := 0; i < len(s) - 1; i++ {
        if value[string(s[i])] < value[string(s[i + 1])] {
            result -= value[string(s[i])]
        } else {
            result += value[string(s[i])]
        }
    }
    result += value[string(s[len(s) - 1])]
    
    return result
}
```

这是12题的姊妹题，将罗马文转换成数字。
这里主要考虑：
1. 当前的罗马文数值小于后一位时，是需要减去的
2. 最后一位肯定是要加上的，所以循环到 **len(s)-2** 就可以了。
