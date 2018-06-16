---
title: LeetCode-17.Letter Combinations of a Phone Number
date: 2017-09-05 11:30:51
tags:
  - LeetCode
---

```
package leetcode

import "strconv"

//Given a digit string, return all possible letter combinations that the number could represent.

func LetterCombinations(digits string) []string {    
    var res []string
    if len(digits) == 0 {
        return res
    }
    var mapping = []string{"0", "1", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"}
    res = append(res, "")
    for i := 0; i < len(digits); i++ {
        x, _ := strconv.Atoi(string(digits[i]))
        for ; len(res[0]) == i; {
            str := res[0]
            res = res[1:]
            for j := 0; j < len(mapping[x]); j++ {
                res = append(res, str + string(mapping[x][j]))
            }
        }
    }
    
    return res
}
```
这道题是电话号码的字母组合。
参考Discuss中排名最高的 **FIFO queue** 解法。
