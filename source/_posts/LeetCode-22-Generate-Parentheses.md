---
title: LeetCode-22.Generate Parentheses
date: 2017-09-12 13:54:19
tags:
  - LeetCode
---
```
package leetcode

//Given n pairs of parentheses, write a function to generate all combinations of well-formed parentheses.
//
//For example, given n = 3, a solution set is:
//
//[
//  "((()))",
//  "(()())",
//  "(())()",
//  "()(())",
//  "()()()"
//]

func GenerateParenthesis(n int) []string {
    var res []string
    helper(n, n, "", &res)        
    return res
}

func helper(left, right int, out string, res *[]string) {
    if left < 0 || right < 0 || left > right {
        return
    }  
    if left == 0 && right == 0 {
        *res = append(*res, out)
        return
    }
    helper(left - 1, right, out + "(", res)
    helper(left, right - 1, out + ")", res)
}
```

参考 **Grandyang** 的博客：[[LeetCode] Generate Parentheses 生成括号](http://www.cnblogs.com/grandyang/p/4444160.html)
