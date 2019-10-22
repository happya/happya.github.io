---
title: Leetcode之旅|Brainy is the new sexy
date: 2017-09-20 22:25:20
tags:
    - Leetcode
    - algorithm
    - 随笔 
categories:
    - Leetcode

---
---

   做这道题的时候一直在想《神探夏洛克》的S02E01，因为想到最开始学到快速排序的时候的惊艳感：人的智商真的就可以这么简单粗暴地展现出来。然后脑子里一直是当时卷福破解Irene手机密码的那段台词,根本停不下来：

Sentiment is the chemical defect found in the losing side.
Disguise is a self-portrait.
I imagine John Watson thinks that love is a mystery to me, but the chemistry is incredibly simple and very destructive. Thank you for the final proof.

啊啦啦，不能再叨叨这段台词了，不然忍不住要看。反正也是听着这段音乐答的题~~~

## 问题描述
---
75 Sort colors

Given an array with n objects colored red, white or blue, sort them so that objects of the same color are adjacent, with the colors in the order red, white and blue.

Here, we will use the integers 0, 1, and 2 to represent the color red, white, and blue respectively.

Note:
You are not suppose to use the library's sort function for this problem.

## 思路
---
   看到这个问题的时候，首先想到的是最近看的快速排序算法的实现过程，因为感觉惊为天人，并意识到为啥之前斯坦福那个老师说算法让人smart,因为感觉不smart怎么能想出这么惊艳的算法（反正我是无论如何也想不出来的。。。）。

   简而言之，我理解的快速排序就是选择一个主元x（最基本的版本里一般就是最后一个元素），然后对序列里所有元素进行扫描，并用一个指针记录i目前比x小的数（这里假设是要实现升序排列），即i总是指向最新找到的比x小的数，因为每扫描到一个比x小的数，都会将其与i当前指向的数的后一个数交换（因为i指向的是最新的比x小的数，所以它后一位肯定比x大），并将i后移一位，最后将序列划分了两堆，比x大的都在一边，比x小的都在另一边。然后再运用divide and conquer的方法，就能实现在O(nlgn)的时间内对序列进行原址排序。
 
<!-- more -->
   为什么做这道题的时候想到了快速排序呢，就是觉得思路是一样的。不同的是，这里我们要提供两个指针，一个指向目前所有发现的0的下一位，一个指向目前发现的1的下一位。然后我们就可以这么操作：
   - 两个指针p0，p1都初始化为0，对数组进行扫描；
   - 如果发现数是0，那么，0的个数要加1，因为0排在最前面，牵一发而动全身，p0要不指向的是当前数，要不指向的就是拍在它后面的1,所以p0当前指向的数变成0后，p1指向的数就应该与p0指向的数进行交换，而目前扫描到的数又应该和p1指向的数进行交换。这和快速排序交换元素的思路其实是一样的：我们并不关心交换哪两个具体的元素，只关心交换的哪两类的具体元素，因为我们对每类元素的具体细节没有要求，而只对它们整体的共性有要求。对快速排序而言，这个共性是比x大或者小这两种情况，而不需要关心它们之间具体谁大谁小；而对这个题目而言则是0，1，2这三类。然后两个指针皆往后移一位；
   - 如果发现数是1则更更简单，只需要将当前指针指向的数变成p1指向的数交换，而p1指向的数变成1即可；
   - 如果发现数是2，则不需要进行任何变化。


   这么说了之后，代码就很容易实现了：


```python
def sortColors(nums):
        """
        :type nums: List[int]
        :rtype: void Do not return anything, modify nums in-place instead.
        """
        p0=p1=0
        for p in range(len(nums)):
            value = nums[p]
            if value==0:
                nums[p] = nums[p1]
                nums[p1] = nums[p0]
                nums[p0] = 0
                p0+=1
                p1+=1
            if value==1:
                nums[p]=nums[p1]
                nums[p1] = 1
                p1+=1
        print(nums)
```

我们测试一下：


```python
sortColors([1,0,2,1,0,1,2,0,2,1,2])
```

    [0, 0, 0, 1, 1, 1, 1, 2, 2, 2, 2]


## 进一步简化代码
---
由上述代码我们可以看到，其实当扫描到0或者1之后，其实有些操作是一样的，比如p1都会后移一位，而p1指向的数都会改变：变成1或者p0当前指向的数，且p0当前指向的数只有两种可能：当前扫描到的数或者其实就是1。同理，p1当前指向的数也只有两种可能：当前扫描到的数或者2。基于这些事实，我们可以对代码进行简化：
- 把扫描到当前位置的值设为2；
- 如果当前位置的实际值小于2（即0或者1），那p1肯定是要后移一位，且p1当前指向的数可能变为1，因为如果是0，所以1到往后挪一位，那p1指向1；如果是1，那1的个数加1，当前指向值也只能是1；
- 如果当前位置的实际值为0，那把p0指向的值变为0即可，然后p0后移一位；
- 如果是2那再好不过，我们已经在第一步处理好了。


```python
def sortColors1(nums):
        """
        :type nums: List[int]
        :rtype: void Do not return anything, modify nums in-place instead.
        """
        p0=p1=0
        for p in range(len(nums)):
            value = nums[p]
            nums[p] = 2
            if value<2:
                nums[p1] = 1
                p1+=1
            if value==0:
                nums[p0]=0
                p0+=1
        print(nums)
```

测试同样通过：


```python
sortColors1([1,0,2,1,0,1,2,0,2,1,2,0,1,0,2])
```

    [0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 2, 2, 2, 2, 2]