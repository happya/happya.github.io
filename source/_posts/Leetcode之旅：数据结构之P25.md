---
title: Leetcode之旅|链表啊，链表！ (0)
date: 2018-01-15 21:06:37
tags: 
    - Leetcode
    - algorithm
    - Data Structure
catogories:
    - Leetcode
mathjax: true
---

　　由于近期状态不太好，很久没更新博客了，惭愧三秒钟。

　　今天放纵自我刷题刷了半天，本来怂怂的我只敢挑战“easy”的题，后来刷了几道linked list的题后，胆儿肥了，刷了道hard模式的题，折腾了将近两个小时，才Pass，现在一把辛酸泪地把思路给记录下来，免得忘记。

  Problem 25: Reverse Nodes in k-Group

  Description:

  ---

Given a linked list, reverse the nodes of a linked list k at a time and return its modified list.

k is a positive integer and is less than or equal to the length of the linked list. If the number of nodes is not a multiple of k then left-out nodes in the end should remain as it is.

You may not alter the values in the nodes, only nodes itself may be changed.

Only constant memory is allowed.

For example,
```python
Given this linked list: 1->2->3->4->5

For k = 2, you should return: 2->1->4->3->5

For k = 3, you should return: 3->2->1->4->5
```

　　思路如下：

　　做这道题由于代码比之前easy模式的长了很多，为了debug方便，我全程是在ipython notebook里面调试的。这道题是和单向链表(singly-linked list)相关的，为了便于调试，我们先定义好这个数据结构，以及如何初始化和可视化一个链表。简单来说，初始化就是将一个List转化为链表，可视化就是把一个链表按head->tail完整的打印出来。



```python
# Definition for singly-linked list.
# 即定义一个单向链表里的节点，它有两个属性：节点表示的数值和指向的下一个节点
class ListNode(object):
     def __init__(self, x):
            self.val = x
            self.next = None
```
```python
# 从一个list生成一个单向链表
def formList(nums):
    if not nums:
        return None
    head = ListNode(nums[0])
    dummy = head
    for num in nums[1:]:
        head.next = ListNode(num)
        head = head.next
    return dummy

# 将一个单向链表从head到tail将其节点数值打印出来
# 这里是为了后面调试方便
def showList(head):
    res = []
    cur = head
    while cur:
        res.append(cur.val)
        cur = cur.next
    return res
```
 <!-- more -->

　　
　　审题: 对于一个有 $n$ 个节点的链表，给定一个 $k$ 值，需要将每块长度为 $k$ 的子链表反向； 如果 $n%k != 0$ ，则对最后一个节点少于 $k$ 的子链表来说，不将其反向。

　　
　　因此有如下几点考虑：
　　1. 如果链表为空或者$k<2$，或者$n<k$，则直接返回head即可；但是由于链表和list不同，不能直接取出长度，所以第三种情况需要另外考虑；
　　2. 对于每个长度为$k$的子链表，可以考虑写一个子程序将其反向，对于每个节点，我们标记其前驱节点pre,和后续节点_next，逐个反向，得到新的head和tail；需要注意的是每个子链表的第一个和最后一个节点需要和前面和后面的子链表关联好；
　　3. 对于第一个反向的子链表，其新的head是特别的，因为就是需要返回的新链表的head。
　　

　　

  子程序如下：

```python
# 传入的参数有三个：
# cur_head: 需要反向的当前子链表的表头
# pre: 当前子链表的前驱节点
# k: 子链表节点个数
# 返回的参数也有三个：
# new_head: 反向后子链表的表头，需要定义为pre的后继
# cur_head:更新cur_head，即需要反向的下一个子链表的表头
# cur_tail: 反向后子链表的最后一个节点，是下一个需要反向的子链表的pre

def rever(cur_head, pre,k):
    new_head = cur_head
    # 对应第一点说的第三种情况，如果子链表长度不足k，则不需要反向
    # 反之，next k-1 次后即是新的表头
    for i in range(k-1):
        new_head = new_head.next
        if not new_head:
            return
    
    cur_node = cur_head
    for i in range(k):
        
       # print('cur_node=',cur_node.val)
        _next = cur_node.next
        cur_node.next = pre
        pre = cur_node
        cur_node = _next
        
    cur_tail = cur_head
    cur_head = cur_node
    cur_tail.next = cur_head


    return new_head,cur_head,cur_tail  
```

　　主程序如下：
```python
def reverseKGroup(head, k):
        """
        :type head: ListNode
        :type k: int
        :rtype: ListNode
        """
        if not head or k<2:
            return head
        cur_head = head # 第一个cur_head就是链表的head
        
        first = rever(cur_head,None,k) # 第一次反向的子链表很特殊
        if not first:
            return head
        _head,cur_head,cur_tail = first[0],first[1],first[2]
        
        
        while(cur_head):
            res = rever(cur_head,cur_tail,k)
           
            if res:
                cur_tail.next = res[0]
                cur_head = res[1]
                cur_tail = res[2]
            else:
                return _head
        return _head   
```

　　测试：
```python
a = formList([1,2,3,4,5,6,7,8,9])
b = formList([1,2,3,4,5,6,7,8,9])
print('a = b = ', showList(a))
res_a = reverseKGroup(a,3)
res_b = reverseKGroup(b,4)
print('res_a = ',showList(res_a))
print('res_b = ',showList(res_b))
```

　　输出：
```python
a = b =  [1, 2, 3, 4, 5, 6, 7, 8, 9]
res_a =  [3, 2, 1, 6, 5, 4, 9, 8, 7]
res_b =  [4, 3, 2, 1, 8, 7, 6, 5, 9]
```

　　通过!!!
　　当看到出现绿色的"Accept"时，第一次觉得绿色这么可爱。。。话不多说了，再接再厉吧，下次刷了Hard模式的再更~
