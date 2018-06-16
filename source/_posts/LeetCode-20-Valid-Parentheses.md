---
title: LeetCode-20.Valid Parentheses
date: 2017-09-08 10:14:53
tags:
  - LeetCode
---

```
package leetcode

//Given a string containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid.
//
//The brackets must close in the correct order, "()" and "()[]{}" are all valid but "(]" and "([)]" are not.

func IsValid(s string) bool {
    var stack []string
    
    for i := 0; i < len(s); i++ {
        tmp := string(s[i])
        length := len(stack)
        switch tmp {
            case "(" : {
                stack = append(stack, tmp)
            }            
            case "{" : {
                stack = append(stack, tmp)
            }            
            case "[" : {
                stack = append(stack, tmp)
            }
            case ")" : {
                if length >0 && stack[length - 1] == "(" {
                    stack = stack[:length - 1]
                } else {
                    return false
                }
            }
            case "}" : {
                if length >0 && stack[length - 1] == "{" {
                    stack = stack[:length - 1]
                } else {
                    return false
                }
            }
            case "]" : {
                if length >0 && stack[length - 1] == "[" {
                    stack = stack[:length - 1]
                } else {
                    return false
                }
            }
        }
    }
    
    return len(stack) == 0
}
```

**stack**
