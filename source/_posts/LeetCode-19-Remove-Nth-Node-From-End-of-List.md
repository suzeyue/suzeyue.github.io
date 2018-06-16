---
title: LeetCode-19.Remove Nth Node From End of List
date: 2017-09-07 11:31:49
tags:
  - LeetCode
---

```
package leetcode

//Given a linked list, remove the nth node from the end of list and return its head.
//
//For example,
//
//   Given linked list: 1->2->3->4->5, and n = 2.
//
//   After removing the second node from the end, the linked list becomes 1->2->3->5.
//Note:
//Given n will always be valid.
//Try to do this in one pass.


type ListNode struct {
    Val int
    Next *ListNode
}

func RemoveNthFromEnd(head *ListNode, n int) *ListNode {
    current := head
    tail := head
    
    for i := 0; i < n && tail != nil; i++ {
        tail = tail.Next
    }
    if tail == nil {
        head = head.Next
        return head
    }       
    for ; tail.Next != nil; {
        tail = tail.Next
        current = current.Next
    }
    current.Next = current.Next.Next
    
    return head
}
```

通过双指针，间隔 **n** ，当后指针到链表尾时，前一个指针就是待删除的节点。
为删除这个节点，我们需要定位这个节点的前驱。
注意删除头节点的情况。
