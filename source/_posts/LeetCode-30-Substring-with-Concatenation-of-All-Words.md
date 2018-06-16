---
title: LeetCode-30.Substring with Concatenation of All Words
date: 2017-11-03 10:17:34
tags:
  - LeetCode
---
### Description
You are given a string, s, and a list of words, words, that are all of the same length. Find all starting indices of substring(s) in s that is a concatenation of each word in words exactly once and without any intervening characters.

For example, given:
s: `"barfoothefoobarman"`
words: `["foo", "bar"]`

You should return the indices: `[0,9]` .


这题要求给出包含列表里所有字符串的子串的下标。
从题中可以知道list中的字符串的长度都是相等的，那么对于字符串s的扫描，我们只需要按照word的长度，这样就可以包含所有的情况。例：s为 `abcdefghi` ，word的长度为 `3` ，那么情况就是 `abc` , `def` , `ghi` , `bcd` , `efg` , `cde` , `fgh` .
另外需要两个map（wordMap、curMap）、变量count和变量left。第一个map用来存放list中的words，第二个map中则放着当前循环中取出的并且存在于wordMap中的字符串；count用来记录当前子串中的word数量；left为子串的下标。
具体思路：
1. 将list中的word存放到wordMap中，vlaue为key出现的次数。
2. 以word的长度为步长，获取slice（即word）。如果此slice在wordMap中，将此slice置入curMap。如果此slice的value大于wordMap中的value，这就代表此时的字符串有多于word，将之前第一次出现的剔除；如果不在，则将count置零，将left后移。
3. 如果此时count的值等于list的长度，那么代表找到一个解。将下表left放入列表中待返回后，将left后移一位，curMap中以left为起始的word计数减一，count减一，继续循环。

参考：[findingea](https://segmentfault.com/a/1190000002625580)

### Code
```
func findSubstring(s string, words []string) []int {    
    if len(s) == 0 || len(words) == 0 {
        return []int{}
    }
    
    var result []int
    wordMap := make(map[string]int)
    strLength := len(s)
    wordLength := len(words[0])
    for _, word := range words {
        wordMap[word]++
    }
    for i := 0; i < wordLength; i++ {
        curMap := make(map[string]int)
        count, left := 0, i
        for j := i; j <= strLength - wordLength; j += wordLength {
            curStr := string(s[j:j + wordLength])
            if _, ok := wordMap[curStr]; ok {
                curMap[curStr]++
                if curMap[curStr] <= wordMap[curStr] {
                    count++
                } else {
                    for true {
                        tmp := string(s[left:left + wordLength])
                        curMap[tmp]--
                        left += wordLength
                        if curStr == tmp {
                            break
                        } else {
                            count--
                        }
                    }
                }
                if count == len(words) {
                    result = append(result, left)
                    tmp := string(s[left:left + wordLength])
                    curMap[tmp]--
                    left += wordLength
                    count--
                }
            } else {
                count = 0
                left = j + wordLength
                curMap = make(map[string]int)
            }
        }
    }
    
    return result
}
```
