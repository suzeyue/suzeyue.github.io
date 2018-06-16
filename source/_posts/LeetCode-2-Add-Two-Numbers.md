---
title: LeetCode-2.Add Two Numbers
date: 2017-08-16 10:20:43
tags:
  - LeetCode
---

```
package leetcode


//You are given two non-empty linked lists representing two non-negative integers.
//The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.
//You may assume the two numbers do not contain any leading zero, except the number 0 itself.

//Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
//Output: 7 -> 0 -> 8


type ListNode struct {
    Val int
    Next *ListNode
}

func AddTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
    result := new(ListNode)
    p := result
    decade := 0
    
    for l1 != nil && l2!= nil {
        sum := decade + l1.Val + l2.Val
        p.Next = &ListNode{Val: sum%10}
        decade = sum / 10
        p = p.Next
        l1 = l1.Next
        l2 = l2.Next
    }
    
    for l1 != nil {
        sum := decade + l1.Val
        p.Next = &ListNode{Val: sum%10}
        decade = sum / 10
        p = p.Next
        l1 = l1.Next
    }
    
    for l2 != nil {
        sum := decade + l2.Val
        p.Next = &ListNode{Val: sum%10}
        decade = sum / 10
        p = p.Next
        l2 = l2.Next
    }
    
    if decade > 0 {
        p.Next = &ListNode{Val: decade}
    }
    
    return result.Next
}
```

1. 进位
2. 当两个链表长度不一致
3. 当处理完最后一个节点，但仍需进一位
