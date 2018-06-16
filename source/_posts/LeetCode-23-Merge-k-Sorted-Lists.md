---
title: LeetCode-23.Merge k Sorted Lists
date: 2017-10-25 15:17:25
tags:
  - LeetCode
---
### Description

** Merge k sorted linked lists and return it as one sorted list. Analyze and describe its complexity. **

#### 一、分治
将列表中的链表对半划分，两两合并。然后再划分合并。

```
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func mergeKLists(lists []*ListNode) *ListNode {

	if len(lists) == 0 {
		return nil
	}

	n := len(lists)
	for ; n > 1; {
		k := (n + 1) / 2
		for i := 0; i < n / 2; i++ {
			lists[i] = mergeTwoLists(lists[i],lists[i + k])
		}
		n = k
	}

	return lists[0]
}

func mergeTwoLists(l1, l2 *ListNode) *ListNode {

    head := new(ListNode)
	cur := head
	for ; l1 != nil && l2 != nil ;  {
		if l1.Val < l2.Val {
			cur.Next = l1
			l1 = l1.Next
		} else {
			cur.Next = l2
			l2 = l2.Next
		}
		cur = cur.Next
	}
	if l1 != nil {
		cur.Next = l1
	}
	if l2 != nil {
		cur.Next = l2
	}

	return head.Next
}
```

#### 二、优先队列
(待补充)


