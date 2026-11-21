# [Problem 3331: Find Subtree Sizes After Changes](https://leetcode.com/problems/find-subtree-sizes-after-changes/description/?envType=daily-question)

## Initial thoughts (stream-of-consciousness)

We have a tree rooted at node 0, and we need to simultaneously change parents for all nodes. For each node x, we find the closest ancestor y with the same character, and if found, x's new parent becomes y.

Key observations:
1. Changes are **simultaneous** - all new parents are determined from the **original** tree structure
2. After changes, the tree remains valid (no cycles since y is always an ancestor of x)
3. We need subtree sizes in the final tree

The naive approach of walking up from each node could be O(n²) for a line tree. We need O(n).

Idea: DFS from root, maintaining a stack per character of nodes on the current path. When entering node x, the top of s[x]'s stack is the closest ancestor with the same character. This gives us the new parent in O(1) per node.

Then build the new tree and compute subtree sizes with another DFS.

## Refining the problem, round 2 thoughts

- "Ancestor" means proper ancestor (not the node itself), confirmed by examples
- Need iterative DFS to avoid recursion depth issues with n up to 10⁵
- For root (node 0), parent stays -1
- If no matching ancestor, keep original parent
- Time: O(n), Space: O(n)

## Attempted solution

```python
from typing import List

class Solution:
    def findSubtreeSizes(self, parent: List[int], s: str) -> List[int]:
        n = len(parent)
        
        # Build original tree adjacency list
        children = [[] for _ in range(n)]
        for i in range(1, n):
            children[parent[i]].append(i)
        
        # Step 1: Find new parent for each node
        # Changes are simultaneous, so we use the original tree structure
        # For each node, closest ancestor with same character becomes new parent
        new_parent = [-1] * n
        char_stacks = [[] for _ in range(26)]  # stack for each lowercase letter
        
        # Iterative DFS to find new parents
        # Stack contains (node, state) where state 0 = entering, 1 = exiting
        dfs_stack = [(0, 0)]
        while dfs_stack:
            node, state = dfs_stack.pop()
            if state == 0:
                # Entering node
                char_idx = ord(s[node]) - ord('a')
                
                # Closest ancestor with same character is top of stack
                if char_stacks[char_idx]:
                    new_parent[node] = char_stacks[char_idx][-1]
                else:
                    new_parent[node] = parent[node]  # keep original (root stays -1)
                
                # Push onto character stack
                char_stacks[char_idx].append(node)
                
                # Push exit event, then children (reversed for correct order)
                dfs_stack.append((node, 1))
                for child in reversed(children[node]):
                    dfs_stack.append((child, 0))
            else:
                # Exiting node - pop from character stack
                char_idx = ord(s[node]) - ord('a')
                char_stacks[char_idx].pop()
        
        # Step 2: Build new tree from new parents
        new_children = [[] for _ in range(n)]
        for i in range(1, n):
            new_children[new_parent[i]].append(i)
        
        # Step 3: Compute subtree sizes using iterative post-order DFS
        answer = [0] * n
        dfs_stack = [(0, 0)]
        while dfs_stack:
            node, state = dfs_stack.pop()
            if state == 0:
                dfs_stack.append((node, 1))
                for child in reversed(new_children[node]):
                    dfs_stack.append((child, 0))
            else:
                size = 1
                for child in new_children[node]:
                    size += answer[child]
                answer[node] = size
        
        return answer
```

- **Approach**: 
  1. Build the original tree from the parent array
  2. Use iterative DFS with per-character stacks to find each node's new parent (closest ancestor with same character in O(1) amortized)
  3. Construct the new tree from computed parents
  4. Compute subtree sizes via post-order DFS

- **Time Complexity**: O(n) - each node is visited a constant number of times
- **Space Complexity**: O(n) - adjacency lists, stacks, and answer array

- **Key insight**: Since all changes happen simultaneously based on the original tree, we can determine all new parents in a single DFS pass using character stacks that track the path from root to current node.