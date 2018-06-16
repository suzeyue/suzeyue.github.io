---
title: LeetCode-28.Implement strStr()
date: 2017-10-30 13:38:09
tags:
  - LeetCode
---
### Description
Implement strStr().
Returns the index of the first occurrence of needle in haystack, or -1 if needle is not part of haystack.

### code
```
func strStr(haystack string, needle string) int {
    haystackLength := len(haystack)
    needleLength := len(needle)
    if needleLength == 0 {
        return 0
    }
    if haystackLength < needleLength {
        return -1
    }
    
    for i := 0; i <= (haystackLength - needleLength); i++ {
        if haystack[i] == needle[0] {
            tmp := string(haystack[i:i + needleLength])
            if strings.Compare(tmp, needle) == 0 {
                return i
            }
        }
    }
    
    return -1
}
```
