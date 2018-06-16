---
title: LeetCode-14.Longest Common Prefix
date: 2017-08-30 20:03:36
tags:
  - LeetCode
---

```
package leetcode

import "math"

//Write a function to find the longest common prefix string amongst an array of strings.

func longestCommonPrefix(strs []string) string {
    if len(strs) == 0 {
        return ""
    }
    
    minLen := len(strs[0])
    for i := 1; i < len(strs); i++ {
        current := len(strs[i])
        minLen = int(math.Min(float64(minLen), float64(current)))
        j := 0
        for ; j < minLen; j++ {         
            if strs[0][j] != strs[i][j] {    
                break
            }
        }
        if minLen > j {
            minLen = j
        }
    }
    
    return string(strs[0][:minLen])
}
```
