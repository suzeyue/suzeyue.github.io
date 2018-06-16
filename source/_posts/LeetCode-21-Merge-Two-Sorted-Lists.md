---
title: LeetCode-21.Merge Two Sorted Lists
date: 2017-09-11 13:24:06
tags:
  - LeetCode
---

```
package leetcode

//Merge two sorted linked lists and return it as a new list.
//The new list should be made by splicing together the nodes of the first two lists.

type ListNode struct {
    Val int
    Next *ListNode
}

func MergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
     
    head := new(ListNode)
    curr := head
    for ; l1 != nil && l2 != nil; {
        if l1.Val < l2.Val {
            curr.Next = l1
            l1 = l1.Next
            curr = curr.Next
        } else {
            curr.Next = l2
            l2 = l2.Next
            curr = curr.Next
        }
    }
    if l1 != nil {
        curr.Next = l1
    }
    if l2 != nil {
        curr.Next = l2
    }
    
    return head.Next
}
```
