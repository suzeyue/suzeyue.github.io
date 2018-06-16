---
title: LeetCode-32.Longest Valid Parentheses
date: 2017-11-06 11:53:54
tags:
  - LeetCode
---
### Description
Given a string containing just the characters `'('` and `')'` , find the length of the longest valid (well-formed) parentheses substring.
For `"(()"` , the longest valid parentheses substring is `"()"` , which has length = 2.
Another example is `")()())"` , where the longest valid parentheses substring is `"()()"` , which has length = 4.

利用stack来处理，另外还有动态规划的方法。

### Code

#### stack
```
func longestValidParentheses(s string) int {
    strLen := len(s)
    if strLen <= 1 {
        return 0
    }
    
    var stack []int
    var index, result int
    for k, v := range s {
        if v == '(' {
            stack = append(stack, k)
        }
        if v == ')' {
            if len(stack) == 0 {
                index = k + 1
            } else {
                stack = stack[:len(stack) - 1]
                if len(stack) == 0 {
                    result = int(math.Max(float64(result), float64(k - index + 1)))
                } else {
                    result = int(math.Max(float64(result), float64(k - stack[len(stack) - 1])))
                }
            }            
        }
    }
    
    return result
}
```

#### 动态规划
（待补充）
