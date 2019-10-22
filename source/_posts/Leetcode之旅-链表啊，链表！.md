---
title: Leetcode之旅|链表啊，链表！(1)
date: 2018-01-17 22:12:23
tags:
    - Leetcode
    - Data Structure
    - 随笔
mathjax: true
---
　　自从前两天刷了那道链表的题之后觉得很有意思，奇妙的数据结构哇咔咔 ~ ~ ~ 但毕竟不是科班出身，对它的一些操作非常不熟悉，每次被各种head,tail,pre,next指来指去的东西绕得晕头转向。嗯，熟能生巧，所以我最近专注于刷链表相关的题目啊哈哈 ~ ~ ~

## 题目描述

　　这篇博客回顾的题目是删除链表中重复元素的问题（链表均已排好序）：

　　基础版：只删除重复了的元素，即保证唯一性 （[Leetcode Problem 83](https://leetcode.com/problems/remove-duplicates-from-sorted-list/) ）。

　　进阶版：删除有重复出现过的元素，即如果元素重复了，就将其从链表中剔除 （[Leetcode Problem 82](https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii) ）。

　　发现我的语言很苍白，还是看实例吧：

　　基础包：

```python
Given 1->2->3->3->4->4->5, return 1->2->5.
Given 1->1->1->2->3, return 2->3.
```

　　升级包：

```python
Given 1->2->3->3->4->4->5, return 1->2->5.
Given 1->1->1->2->3, return 2->3.
```

## 解题思路及相关代码

### 基础版

　　对于基础包而言，我们只需要一个指针，即下面代码中的cur，从head开始挨个遍历链表节点（所以链表已经排序好这个前提很重要），对于每个cur，将其与其后继的节点值比较，如果相同，说明这个后继需要删除，对链表而言，操作则相当简单，直接将cur的后继改为当前后继的后继即可。如果不同，说明没有与当前节点值重复的了，直接将当前节点后移即可，即将cur更新为当前节点的后继。
　　代码如下：


```python
def deleteDuplicates(head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        # 链表为空的特殊情况
        if not head:
            return head
        # 当前节点初始化为表头
        cur = head
        
        # 遍历，循环条件为当前节点和当前节点的后继不为空
        while cur and cur.next:
            # 如果当前节点值和其后继值相等，则将其后继改为后继的后继
            if cur.val == cur.next.val:
                cur.next = cur.next.next
            # 如果不相等，则将当前节点更新
            else:
                cur = cur.next
        return head
```

 <!-- more -->

 
### 进阶版

　　和基础版相比，进阶版的难度体现在两个方面：第一，因为重复出现过的元素需要删除，所以表头可能会被修改（如第一次出现的元素就重复的情况）；第二，因为重复出现过的元素很随机，所以一个指针不够提供足够的信息，比如上述例子中的第一个，在将3和4删除后，得知道5的前驱是2。
　　对此，我的解决方案是：
　　１.　使用两个指针，一个用来指代当前节点(cur)，初始化为表头，另一个为前驱节点(pre)，可以在删除重复节点后，给下一个节点提供前驱信息，初始化为Null。
　　2.　从当前节点开始遍历链表，从简单的情况开始处理，即如果当前值不等于其后继值，那么非常lucky，这个节点不重复了，将前驱节点更新为当前节点，将当前节点更新为其后继节点，然后跳出本次循环，move on；
　　3.　如果相等呢？那我们就得顺着往下捋,即更新后继节点_next，直到不相等了，这时_next指向的是第一个不与前面节点重复的节点，而cur指向的是这批重复节点的第一个节点。那么这时要删除的话就有两种情况了:第一，如果重复段在表头，说明head要更新，更新为_next节点了，这时pre依旧指向Null，然后将cur更新为_next；第二，如果这批重复的是中间的批次，说明head不用更新了，但因为中间这批重复的全删了，所以pre的后继得更新为_next，和第一种情况一样，cur也要后移，即更新为_next。
　　代码如下：


```python
def deleteDuplicates(head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        if not head:
            return head
        
        # 两个指针：pre和cur
        pre = None
        cur = head
        
        # 遍历所有节点
        while cur:
            # 一个临时指针指向cur的后继
            _next = cur.next
            
            if not _next:
                return head
            
            # 如果两个值不相等，则将pre和cur都往后移
            elif cur.val != _next.val:
                pre = cur
                cur = _next
                continue
            
            # 如果相等，则将_next一直捋到不重复为止
            else:
                while _next and _next.val == cur.val:
                    _next = _next.next
                
                # 如果重复段在表头，则需要更新表头信息
                if cur == head:
                    head = _next
                # 如果重复段在中间，则需要将pre的后继更新为_next
                else:
                    pre.next = _next
                # 继续遍历
                cur = _next
        return head
```

## 结果

　　测试：均已通过

## 思考：

　　１.　基础版是用迭代法，“凡是迭代即可递归”，试试递归如何实现？
　　2.　进阶版是最优化的吗？可以逛逛discuss看是否还可进一步优化？