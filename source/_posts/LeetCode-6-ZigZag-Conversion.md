---
title: LeetCode-6.ZigZag Conversion
date: 2017-08-20 15:44:45
tags:
  - LeetCode
---

```
package leetcode

//The string "PAYPALISHIRING" is written in a zigzag pattern on a given number of rows like this: (you may want to display this pattern in a fixed font for better legibility)
//
//P   A   H   N
//A P L S I I G
//Y   I   R
//And then read line by line: "PAHNAPLSIIGYIR"
//Write the code that will take a string and make this conversion given a number of rows:
//
//string convert(string text, int nRows);
//convert("PAYPALISHIRING", 3) should return "PAHNAPLSIIGYIR".

func Convert(s string, numRows int) string {
    if numRows <= 1 {
        return s
    }
    
    var result []byte
    size := 2 * numRows - 2
    for i := 0; i < numRows; i++ {
        for j := i; j < len(s); j += size {
            result = append(result, s[j])
            if i != 0 && i != numRows - 1 && j + size - 2*i < len(s) {
                result = append(result, s[j + size - 2*i])
            }
        }
    }
    
    return string(result)
}

```

这道题就是找规律。 **2*numRows-2** 是关键，然后就是每行（除首尾行）的 **j + size - 2 * i** 。
