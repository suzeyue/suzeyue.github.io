---
title: LeetCode-10.Regular Expression Matching
date: 2017-08-27 16:34:17
tags:
  - LeetCode
---

```
package leetcode

//Implement regular expression matching with support for '.' and '*'.
//
//'.' Matches any single character.
//'*' Matches zero or more of the preceding element.
//
//The matching should cover the entire input string (not partial).
//
//The function prototype should be:
//bool isMatch(const char *s, const char *p)
//
//Some examples:
//isMatch("aa","a") → false
//isMatch("aa","aa") → true
//isMatch("aaa","aa") → false
//isMatch("aa", "a*") → true
//isMatch("aa", ".*") → true
//isMatch("ab", ".*") → true
//isMatch("aab", "c*a*b") → true

func IsMatch(s string, p string) bool {
    m, n := len(s), len(p)
    dp := make([][]bool, m + 1)
    for i:= 0; i <= m; i++ {
        dp[i] = make([]bool, n + 1)
    }
    dp[0][0] = true
    
    for i := 1; i <= m; i++ {
            dp[i][0] = false
    }
        
    for j := 1; j <= n; j++ {
        dp[0][j] = j > 1 && '*' == p[j - 1] && dp[0][j - 2]
    }        
        
    for i := 1; i <= m; i++ {
        for j := 1; j <= n; j++ {
            if p[j - 1] != '*' {
                dp[i][j] = dp[i - 1][j - 1] && (s[i - 1] == p[j - 1] || '.' == p[j - 1])
            } else {
                dp[i][j] = dp[i][j - 2] || (s[i - 1] == p[j - 2] || '.' == p[j - 2]) && dp[i - 1][j]
            }
        }
    }
    
    return dp[m][n]
}
```
