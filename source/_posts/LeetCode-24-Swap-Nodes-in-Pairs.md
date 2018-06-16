---
title: LeetCode-24.Swap Nodes in Pairs
date: 2017-10-27 16:05:02
tags:
  - LeetCode
---
### Description
** Given a linked list, swap every two adjacent nodes and return its head. **
** For example, **
** Given `1->2->3->4` , you should return the list as `2->1->4->3` . **
** Your algorithm should use only constant space. You may not modify the values in the list, only nodes itself can be changed. **

单链表操作，主要需要记住链表头以及下一对需要交换的node。

```
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func swapPairs(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
    
    var mark = new(ListNode)
    mark.Next = head
    node1, node2 := mark, head
    for ; node2 != nil && node2.Next != nil; {
        tmp := node2.Next.Next
        node2.Next.Next = node2
        node1.Next = node2.Next
        node2.Next = tmp
        node1 = node2
        node2 = node2.Next
    }
    
    return mark.Next
}
```
