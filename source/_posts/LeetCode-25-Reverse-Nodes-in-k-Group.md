---
title: LeetCode-25.Reverse Nodes in k-Group
date: 2017-10-29 17:10:25
tags:
  - LeetCode
---
### Description
**Given a linked list, reverse the nodes of a linked list k at a time and return its modified list.**
**k is a positive integer and is less than or equal to the length of the linked list. If the number of nodes is not a multiple of k then left-out nodes in the end should remain as it is.**
**You may not alter the values in the nodes, only nodes itself may be changed.**
**Only constant memory is allowed.**
**For example,**
**Given this linked list: `1->2->3->4->5` **
**For k = 2, you should return: `2->1->4->3->5` **
**For k = 3, you should return: `3->2->1->4->5` **

同样是翻转链表，不过这次是以k个为一组。
所以这里需要考虑两个问题：

1. k个节点的链表如何翻转
2. 如何将输入链表按k个划分， 同时考虑到最后一组可能不满k个导致无需翻转

翻转的话，依次将2、3、4、... 、k位的节点插到表头；
划分的话有多种，在 [Grandyang](http://www.cnblogs.com/grandyang/p/4441324.html) 的博客里看到了一个非常有意思的划分：通过链表的长度 length 和 k 进行比较，翻转，直至 length < k 结束。
Golang实现：

```
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseKGroup(head *ListNode, k int) *ListNode {
    if head == nil || k == 1 {
        return head
    }
    
    length := 0
    var mark = new(ListNode)
    mark.Next = head
    before := mark
    cur := before.Next
    for ; cur != nil; {
        cur = cur.Next
        length++
    }
    for ; length >= k; {
        cur = before.Next
        for i := 1; i < k; i++ {
            tmp := cur.Next
            cur.Next = tmp.Next
            tmp.Next = before.Next
            before.Next = tmp
            
        }
        before = cur
        length -= k
    }
    
   
    return mark.Next
}
```
