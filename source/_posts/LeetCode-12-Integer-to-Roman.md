---
title: LeetCode-12.Integer to Roman
date: 2017-08-29 11:02:28
tags:
  - LeetCode
---

```
package leetcode

//Given an integer, convert it to a roman numeral.
//Input is guaranteed to be within the range from 1 to 3999.

func IntToRoman(num int) string {
    var roman = []byte{'M', 'D', 'C', 'L', 'X', 'V', 'I'}
    var value = []int{1000, 500, 100, 50, 10, 5, 1,}
    var result []byte
    for i := 0 ; i < 7 ; i += 2 {
        x := num / value[i]
        switch {
            case x == 9: {
                result = append(result, roman[i])
                result = append(result, roman[i - 2])
            }
            case x > 4: {
                result = append(result, roman[i - 1])
                for j := 6; j <= x; j++ {
                    result = append(result, roman[i])
                }
            }
            case x == 4: {
                result = append(result, roman[i])
                result = append(result, roman[i - 1])
            }
            default: {
                for j := 1; j <= x; j++ {
                    result = append(result, roman[i])
                }
            }
        }
        num %= value[i]
    }
    
    return string(result)
}
```

数字转换成罗马文，首先要知道罗马文表示方法：
```
I    V    X    L    C    D    M
1    5    10   50   100  500  1000
```
因为已规定输入的范围：1~3999 ，所以这里没有验证输入的合法性。
同时根据已知范围，可以列出所有情况，时间复杂度为：O（1）
参考：[Grandyang的Blog](http://www.cnblogs.com/grandyang/p/4123374.html)

```
func intToRoman(num int) string {
    var M = [4]string{"", "M", "MM", "MMM"}
    var	C = [10]string{"", "C", "CC", "CCC", "CD", "D", "DC", "DCC", "DCCC", "CM"}
    var	X = [10]string{"", "X", "XX", "XXX", "XL", "L", "LX", "LXX", "LXXX", "XC"}
    var	I = [10]string{"", "I", "II", "III", "IV", "V", "VI", "VII", "VIII", "IX"}
    return M[num / 1000] + C[(num % 1000) / 100] + X[(num % 100) / 10] + I[num % 10]
}
```

