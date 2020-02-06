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
    bool isValid(int x, int y, vector<string>& grid){
        if(x < 0 || y < 0 || x >= grid.size() || y >= grid[0].size() || grid[x][y] == '#')
            return false;
        return true;
    }
public:
    int shortestPathAllKeys(vector<string>& grid) {
        int m = grid.size();
        int n = grid[0].size();
        
        queue<pair<int,int>> q;
        int K = 0;
        // visited to mark the state of owned keys
        // at most keys, using bitmap, 111111 = 2^6 - 1, then total state is 64
        vector<vector<vector<bool>>> visited(m, vector<vector<bool>>(n, vector<bool>(64, 0)));

        for(int i = 0; i < m; i++){
            for(int j = 0; j < n; j++) {
                if(grid[i][j] == '@'){
                    q.push({i*n+j, 0});
                    visited[i][j][0] = true;
                }
                if(grid[i][j] >= 'A' && grid[i][j] <= 'F')
                    K |= (1 << (grid[i][j] - 'A'));
            }
        }
        int d[4][2] = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
        int path = 0;
        while(!q.empty()){
            int sz = q.size();
            for(int i = 0; i < sz; i++){
                int x = q.front().first / n;
                int y = q.front().first % n;
                int ownKeys = q.front().second;
                q.pop();

                // holding all keys
                if (ownKeys == K)
                    return path;

                for(int j = 0; j < 4; j++){
                    int newX = x + d[j][0];
                    int newY = y + d[j][1];
                    int k = ownKeys;
                    if(!isValid(newX, newY, grid))
                        continue;
                    
                    char c = grid[newX][newY];
                    // found key: update k
                    if(c >= 'a' && c <= 'f')
                        k |= (1 << (c - 'a'));
                    
                    // found locker
                    else if(c>= 'A' && c <= 'F' && ((k>>(c - 'A')) & 1) == 0)
                        continue;
                    if(!visited[newX][newY][k]){
                        visited[newX][newY][k] = true;
                        q.push({newX * n + newY, k});
                    }
                }
            }
            
            path++;
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