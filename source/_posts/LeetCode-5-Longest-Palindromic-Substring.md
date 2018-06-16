---
title: LeetCode-5.Longest Palindromic Substring
date: 2017-08-19 19:21:10
tags:
  - LeetCode
---

```
package leetcode

//Given a string s, find the longest palindromic substring in s. You may assume that the maximum length of s is 1000.
//
//Example:
//Input: "babad"
//Output: "bab"
//Note: "aba" is also a valid answer.
//
//Example:\
//Input: "cbbd"
//Output: "bb"

func longestPalindrome(s string) string {
    head, tail := 0, 0
    for i := 0; i < len(s); i++ {
        len1 := expandAroundCenter(s, i, i)
        len2 := expandAroundCenter(s, i, i+1)
        if len1 < len2 {
            len1 = len2
        }
        if len1 > tail - head {
            head = i - (len1 - 1) / 2
            tail = i + len1 / 2
        }
    }
    
    return s[head:tail + 1]
}

func expandAroundCenter(s string, left, right int) int {
    for left >= 0 && right < len(s) && s[left] == s[right] {
        left--
        right++
    }
    return right - left - 1
}
```

#### 思路

根据回文字符串的特点，找到回文字符串的中心然后向两侧扩散。
时间复杂度：O(n^2)

其实还有时间复杂度为O(n)的算法： **Manacher's Algorithm**
[Manacher's ALGORITHM](https://www.felix021.com/blog/read.php?2040)

