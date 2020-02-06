---
title: graph-bfs-with-bitmask
date: 2020-02-05 22:45:52
tags:
- algorithms
- graph theory
---

---
# Graph BFS with bitMask

In ordinary BFS, we always use a `vector` (or array) to mark if a node is visited or not. However, in some problems, the re-visited is allowed, or sometimes we must revisit some nodes to get the solution. In this case, we can use a bitmap, combined with the node number, to create the new node id for the original node. So these new IDs not only contains the node number but also contains information like the path that gets to this node.

### How to construct the new node ID?

1. The most intuitive way is to construct a new class as below.

```text
class Node {
    int v;
    int bitMask;
};
```

Then if we have a set called `visited`, instead of add the original node to `visited`, we insert the new `Node` to it. And to check if one node is visited, we have to check if the corresponding `Node` is visited, which means both the node and the `bitMask` have to be checked.

2. The second way is to utilize the `32bit` feature of integer. If the node number is less than 16 \(and represented by integer in \[0, 15\]\), we can construct the node id as below.

```text
newNode = (node << 16) | (1 << node);
```

### Real Problems

### Problem 1

#### [847. Shortest Path Visiting All Nodes](https://leetcode.com/problems/shortest-path-visiting-all-nodes/)

Solution: BFS + bitMask. Some nodes have to be re-visited because sometimes it is impossible to traverse all the nodes without heading back.

<!-- more -->

```text
class Solution {
   
public:
    int shortestPathLength(vector<vector<int>>& graph) {
        int N = graph.size();
        queue<pair<int,int>> q; // the queue for BFS
        unordered_set<int> visited; // visited record
        for(int i = 0; i < N; i++){
            int bitMask = (1 << i);
            q.emplace(i, bitMask); // both node and bitMask
            visited.insert((i << 16) | bitMask); // new node ID
        }
        int fullMask = (1 << N) - 1;
        int level = 0;
        while(!q.empty()){
            int sz = q.size();
            for(int i = 0; i < sz; i++){
                int v = q.front().first;
                int bitMask = q.front().second;
                q.pop();
                
                if(bitMask == fullMask)
                    return level;
                for(auto& w : graph[v]){
                    // The mask for w is get the mask for v
                    // and add the visited bit for w
                    // that is bitMask | (1 << w)
                    int nextMask = bitMask | (1 << w); 
                    // construct the nodeID with node number
                    // and mask
                    int nodeId = ((w << 16) | nextMask);
                    // If this nodeId is visited,
                    // it will not be consider again
                    if(visited.find(nodeId) != visited.end())
                        continue;
                    q.emplace(w, nextMask);
                    visited.insert(nodeId);
                    
                }
            }
            level++;
        }
        return level;
    }
};
```

#### Problem 2

#### [864: Shortest Path to get all keys](https://leetcode.com/problems/shortest-path-to-get-all-keys/)

Some nodes need to be revisited because sometimes we should first get keys and then get back to open the corresponding lockers.

```text
class Solution {
private:
    bool isValid(vector<string>& grid, int x, int y){
        if(x < 0 || y < 0 || x >= grid.size() || y >= grid[0].size() || grid[x][y] == '#')
            return false;
        return true;
    }
public:
    int shortestPathAllKeys(vector<string>& grid) {
        queue<pair<int,int>> q;
        unordered_set<int> visited;
         
        int m = grid.size() , n = grid[0].size();
        int totalKey = 0;
        for(int i = 0; i < m; i++){
            for(int j = 0; j < n; j++){
                char c = grid[i][j];
                if(c == '@'){
                    int pos = i * n + j;
                    q.emplace(pos, 0);
                    visited.insert(pos<<16);
                }
                if(c >= 'A' && c <= 'F')
                    totalKey |= (1 << (c - 'A'));
            }
        }
        // get 4 direction vectors
        int d[4][2] = {{1, 0}, {0, -1}, {-1, 0}, {0, 1}};
        int level = 0;
        while(!q.empty()){
            int sz = q.size();
            for(int i = 0; i < sz; i++){
                int pos = q.front().first;
                int bitMask = q.front().second;
                q.pop();
                
                // if hold all keys
                if(bitMask == totalKey)
                    return level;
                
                int x = pos / n, y = pos % n;
                for(int j = 0; j < 4; j++){
                    int newX = x + d[j][0];
                    int newY = y + d[j][1];
                    if(!isValid(grid, newX, newY)) continue;
                    
                    char c= grid[newX][newY];
                    int ownKeys = bitMask;
                    // if it is a key
                    if( c >= 'a' && c <= 'f')
                        ownKeys |= (1 << (c - 'a'));
                    if(ownKeys == totalKey)
                        return level + 1;
                    // if it is  a locker without corresponding key
                    else if(c >= 'A' && c <= 'F' && ((ownKeys >> (c - 'A')) & 1) == 0 )
                        continue;
                    
                    // other condictions: road or locker but has key
                    int newPos = newX * n + newY;
                    int nodeId = ((newPos << 16) | ownKeys);
                    if(visited.find(nodeId) == visited.end()){
                        visited.insert(nodeId);
                        q.emplace(newPos, ownKeys);
                    }
                    
                }
            }
            level++;
        }
        return -1;
    }
};
```

#### Problem 3

#### [403: Frog Jump](https://leetcode.com/problems/frog-jump/)

Similar method. Using bitmap to mark the visiting state of each stone.

```text
class Solution {
    
private:
    unordered_map<long, bool> dp;
    bool helper(vector<int>& stones, int cur, int k){
        long comb = cur;
        comb = (comb << 32) | k;
        if(dp.find(comb) != dp.end())
            return dp[comb];
        
        for(int i = cur + 1; i < stones.size(); i++){
            int gapsize = stones[i] - stones[cur];
            if(gapsize < k - 1) continue;
            if(gapsize > k + 1){dp[comb] = false; return false;};
            if(helper(stones, i, gapsize))
                return dp[comb] = true;
        }
        
        dp[comb] = (cur == stones.size() - 1);
        return dp[comb];
    }
public:
    bool canCross(vector<int>& stones) {
        
        return helper(stones, 0, 0);
    }
};
```