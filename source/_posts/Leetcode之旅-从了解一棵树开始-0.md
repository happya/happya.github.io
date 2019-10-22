---
title: Leetcode之旅|从了解一棵树开始(0)
date: 2018-01-30 00:32:42
tags:
    - Leetcode
    - Data Structure
    - Algorithm
    - Tree
    - 随笔
mathjax: True
---


　　今天心情特别郁闷。
　　没想到会有人觉得我给港科大丢脸了，我简直是震惊。
　　生活不顺已经很艰难，这句话简直字字诛心。
　　我能在如此逆境中保持不断学习的状态，在英语并不差的情况下还想着变得更好，在有马甲线的情况下还能自制锻炼，我哪里差呢？最多就是我没能发出paper。我花了很久的时间让自己相信paper这事其实并不是我不够聪明，也不是不够努力，花费如此气力来承认自己的能力与价值，不能因为别人随随便便的一句话就坍塌。
　　就像《无问西东》里说的：要坚信自己的珍贵。

## 题目描述

 　　从这周开始做了点tree相关的题，发现递归和深度/广度优先搜索(DFS/BFS)是绕不开的话题。对于递归，我还是没能熟练操作，现在只能通过半参考别人的解答半练习的状态来学习这类题目的解法。

 　　今天的主题的Path Sum，就是给出一棵二叉树，一个target数值，找出节点值之和等于target的路径。按照难易程度，分为如下几个level:

 >　　- 1.0版本：给出一棵树，一个target值，从根节点(root)开始到叶子结点(leaves)结束，如果存在一个路径，位于该路径的所有节点的数值之和等于target，说明path存在，返回True;反之，返回False。
 >　　- 2.0版本：在1.0的基础上，找出所有满足的路径，并将每个符合要求的路径的节点值以列表形式返回。
 >　　- 3.0版本： 在上述两个版本的基础上，不限于从root节点开始，也不限于到leaves节点结束，找出所有满足要求的路径数。
 

 　　比如，对如下一棵二叉树和target = 22：

```python

              5
             / \
            4   8
           /   / \
          11  13  4
         /  \      \
        7    2      1
```

> 　　- 1.0版本：返回True，因为存在root-to-leaf路径5->4->11->2，其节点值之和等于target=22。
> 　　- 2.0版本：返回[[5,4,11,2],[5,8,4,5]]。
> 　　- 3.0版本：返回3，因为存在三条路径：
> 　　1. 5->4->11->2;
> 　　2. 4->11->7;
> 　　3. 8->13。
> 　　

## 解题思路

### 1.0版本

　　从二叉树的形状就可以看出，和tree相关的题目会很直观地想到递归，因为从根节点开始，以它的左节点或者右节点(存在且不为leaf)为根节点的话依然是一棵树。同时，这道题也很容易让人想到leetcode的入门第一题：twosum。所以这道题我们可以按照如下思路递归：
> 　　- 根节点为null：返回False;
> 　　- 根节点不为null且存在左/右:递归调用hasPathSum函数，其中传递参数root改为root.left和root.right，target改为target-root.val,左右的返回值逻辑为or (递推关系);
> 　　- 如果根节点值等于target值，说明路径存在，返回True(出口条件)。

　　代码如下：

### 1.0版本代码
---
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def hasPathSum(self, root, target):
        """
        :type root: TreeNode
        :type sum: int
        :rtype: bool
        """
        if not root:
            return False
        if root.left or root.right:
            return self.hasPathSum(root.left,target-root.val) or self.hasPathSum(target.right,sum-root.val)
        return root.val==sum
        
```


### 2.0版本

　　这道题我一开始就否认了直接递归，觉得很难写，后来看到discuss说是用DFS，觉得挺好。下面的解析思路和代码希望我能学习并运用好。
　　简而言之，利用DFS，所以本质上还是递归的，递归思路如下：
> - 设定全局变量res以存储结果，而且dfs函数是没有返回值的，只是在递归调用过程中对res进行修改，最后在主程序中返回。这种情况其实可以直接在主函数里将dfs写成方法。另外，只在根节点不为null的时候，dfs函数会有操作。
> - dfs传递的参数有root,target,path,res,其中root,target会在递推关系中更改，而path值会记录访问路径。
> - 当节点值等于target且它没有左右孩子时(说明为leaf)，说明该路径满足条件，则将该节点加入路径，而该路径加入res。
> - 递推关系：root->root.left/root.right,target->target-root.val，path->path+[root.val],res按实际情况决定是否更新。

<!--more-->
　　代码如下：

### 2.0版本代码
```python
class Solution:
    def pathSum1(self, root, target):
        """
        :type root: TreeNode
        :type sum: int
        :rtype: List[List[int]]
        """
        if not root:
            return []
        res = []
        self.dfs(root,target,[],res)
        return res
    
    def dfs(self,root,target,path,res):
        if root:
            if root.val==target and not root.left and not root.right:
                res.append(path+[root.val])
            self.dfs(root.left,target-root.val,path+[root.val],res)
            self.dfs(root.right,target-root.val,path+[root.val],res)
```

　　记得当时学算法课的时候，dfs和bfs都是通过stack实现的，所以我们可以不需要递归，而是借助stack来实现。这样算法的效率明显提升(时间缩短20%)。
　　为啥stack可以实现呢？因为我们用堆栈不仅记录的节点，同时记录了当前节点的历史访问路径。而当我们每访问一个节点，只需要将其入栈，且在其之前访问过的节点已经计入了path。而当检验路径是否满足需求时，节点和以该节点为终点的路径会出栈。如果符合，则纳入res；如果不符合，则继续往下走(深度优先)。
　　发现这段写得不太好，可能是因为我理解得还不够透彻，接下来这个部分要加强练习。

### 2.0 stack代码
```python
class Solution:
    def pathSum2(self, root, target):
        """
        :type root: TreeNode
        :type sum: int
        :rtype: List[List[int]]
        """
        if not root:
            return []
        res = []

        # 用stack记录当前根节点，和值，以及路径
        stack = [(root,root.val,[root.val])]
        while stack:
            cur, cur_sum,cur_path = stack.pop()
            if not cur.left and not cur.right and cur_sum==target:
                res.append(cur_path)
            if cur.left:
                stack.append((cur.left,cur_sum+cur.left.val,cur_path+[cur.left.val]))
            if cur.right:
                stack.append((cur.right,cur_sum+cur.right.val,cur_path+[cur.right.val]))
        return res
            
```


### 3.0版本

　　3.0版本很有意思，竟然是easy级别的，但我用自己想到的方法跑太慢了，时间复杂度感觉应该是$O(N^2)$，所以跑了将近1s，果不其然是垫底级别的。后来看到别人的dfs方法时间瞬间缩短到了95ms，应该是hash的作用吧(小白不确定是不是这个原因？)。
　　先来看> 1000 ms的解法：运用了1.0版本的找路径的方法，不过因为现在不限于是root-to-leaf，所以得遍历所有的节点将其作为root，找到即计数加1。

### 3.0 慢速版代码
---

```python
class Solution:
    def pathSum3(self, root, target):
        """
        :type root: TreeNode
        :type sum: int
        :rtype: int
        """
        if root:
            return self.hasPathSum(root,s)+self.pathSum(root.left,s)+self.pathSum(root.right,s)
        return 0
            
    
    def hasPathSum(self,root,target):
        if root:
            return int(root.val==s)+self.hasPathSum(root.left,s-root.val)+self.hasPathSum(root.right,s-root.val)
        return 0
```

　　加速版解法很有意思，采用的字典dict这一数据结构，用来存储当前访问路劲节点的和值。判断的方法也机智(真是没想到)，即将每次采集到的和值减去target，如果这个键值存在，说明该键值存储的路径的终点到目前节点的"距离"即为target (再一次体会到智商被虐的痛楚)。
　　这版本解法的另一特点是运用dfs，可见以后解题得形成这种思维习惯：什么样类型的问题可以化为dfs的问题？

### 3.0加速版代码
```python
class Solution:
    def pathSum4(self, root, target):
        """
        :type root: TreeNode
        :type sum: int
        :rtype: int
        """
        if not root:
            return 0
        # 属性值，记录符合要求的路径数目，可以在dfs中更改
        self.count = 0

        # 记录已访问的路径的和值
        # 键值不存在或0表示不曾出现，为1表示出现过
        # 只有0和1两个值(暂时不明白这么说是否正确)
        dic = {0:1}
        self.dfs(root,s,0,dic)
        return self.count
    
    # dfs的四个参数：
    # node:当前访问节点
    # target：目标值
    # cur_sum：当前路径和值
    # dic:路径和值字典
    def dfs(self,node,target,cur_sum,dic):
        if node:
            # 上一轮和值与当前节点值相加更新为此轮和值
            cur_sum += node.val
            # 如果存在与此轮和值“距离”为target的键值，count+1
            self.count += dic.get(cur_sum-target,0)
            # 将此轮和值存入字典，次数加1
            dic[cur_sum] = dic.get(cur_sum,0)+1  
            # 左右孩子递归
            self.dfs(node.left,s,cur_sum,dic)
            self.dfs(node.right,s,cur_sum,dic)
            # 递归结束后此轮和值次数清空，因为要开始新的路径记录和判断了
            dic[cur_sum]-=1
```

## 总结

　　太虐了！！！不过我感觉为了熟练操作二叉树，首先应该熟悉的是操作这类数据结构的方法，目前我理解的主要是递归和DFS/BFS，所以对应的应该对使用stack(可能还有dic)都能很熟悉，因为很多时候使用stack会出现逻辑衔接不上或者出口甚至是入口条件找不对等等。
　　无论如何，努力吧。
　　嗯。