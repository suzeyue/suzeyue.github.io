---
title: LeetCode-3.Longest Substring Without Repeating Characters
date: 2017-08-17 11:13:59
tags:
  - LeetCode
---

```
package leetcode

//Given a string, find the length of the longest substring without repeating characters.
//
//Examples:
//Given "abcabcbb", the answer is "abc", which the length is 3.
//Given "bbbbb", the answer is "b", with the length of 1.
//Given "pwwkew", the answer is "wke", with the length of 3. Note that the answer must be a substring, "pwke" is a subsequence and not a substring.

func LengthOfLongestSubstring(s string) int {
    max := 0
    head := 0
    
    for index, value := range s {
        subString := s[head: index]
        if strings.Contains(subString, string(value)) {
            head = strings.Index(subString, string(value)) + head + 1
        } else {
            if length := len(subString) + 1; length > max {
                max = length
            }
        }
    }
    
    return max
}
```

第一次提交未通过：
输入： `"bbtablud"`
正确结果： 6
错误输出： 7

回顾了下逻辑，问题出现在对子串起始地址的判断。之前错误代码：
```
head = strings.Index(s, string(value)) + 1
```
之前对子串起始位置选择是通过原始string **s** 来判断的，这样就会出现一个问题：当value在subString之前出现多次时，head只是value第一次出现的位置。
的确是考虑不周。
