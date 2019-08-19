---
layout: post
title: LeetCode Weekly Contest 150
date: 2019-08-18
tags: [leetcode]
comments: true
toc: true
---

### 1160. Find Words That Can Be Formed by Characters

This is a easy level question. Steps of solution:

1. count number of each letter in chars array. Note as ```countChars```.
2. For each word in the array.
   1. Make a copy of countChars. Note as ```copy```.
   2. For each letter in word, do ```copy[leter]--```. if ```copy[letter]``` become less than 0. The word is not a good string.
   3. If good string, add length to answer.

```java
class Solution {
    public int countCharacters(String[] words, String chars) {
        int ans = 0;
        int[] countChars = new int[26];
        for (char c : chars.toCharArray()) countChars[c-'a']++;

        for (String w : words) {
            int[] copy = countChars.clone();
            boolean valid = true;
            for (char c : w.toCharArray()) {
                copy[c-'a']--;

                if (copy[c-'a'] < 0) {
                    valid = false;
                    break;
                }
            }

            if (valid) ans += w.length();
        }

        return ans;
    }
}
```



### 1161. Maximum Level Sum of a Binary Tree

Typical tree problem which can be solved by DFS or BFS. I am using DFS here.

Since we need to sum up values by each level. So we need a list to keep track of the current sums of different levels.

Lets say ``` sums``` is the list. And index i in sums indicate the sum of level i+1 of the tree.

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public int maxLevelSum(TreeNode root) {
        List<Integer> sums = new ArrayList<>();
        search(root, 1, sums);

        int max = Integer.MIN_VALUE;
        int ans = -1;
        for (int i = 0; i < sums.size(); i++) {
            if (sums.get(i) > max) {
                max = sums.get(i);
                ans = i+1;
            }
        }
        return ans;
    }

    private void search(TreeNode curr, int level, List<Integer> sums) {
        if (curr == null) return;

        if (sums.size() < level) {
            sums.add(curr.val);
        } else {
            int sum = curr.val + sums.get(level-1);
            sums.set(level-1, sum);
        }

        search(curr.left, level+1, sums);
        search(curr.right, level+1, sums);
    }
}
```

### 1162. As Far from Land as Possible

For a given water cell `grid[i][j]`. Let `dist[i][j]` denotes its distance to the nearest land cell. 

We can find out that `dist[i][j]`can be calculated by its four neighbors. The expression can be denoted as `dist[i][j] = 1 + Min(dist[i+1][j], dist[i-1][j], dist[i][j-1], dist[i][j+1])`.

Since we need neighbors in four direction to calculater `dist[i][j]`, we can use a two-pass solution to calculate values of all `dist[i][j]`.

In the first pass, we iterate the grid from top-left to bottm-right. And we calculate `dist[i][j]` first based on its top and left neighbors, that is `dist[i-1][j], dist[i][j-1]`.

In the second pass, we iterate the grid from bottom-right to top-left. And then we furhter optimize `dist[i][j]` based on its bottom and right neighbors, that is `dist[i+1][j], dist[i][j+1]`.

In the final step we just need to iterate the whole `dist` and find out the max value which is the answer to return.

```java
class Solution {
  	public int maxDistance(int[][] grid) {
        int m = grid.length;
        int n = grid[0].length;
        
        int[][] dist = new int[m+2][n+2];

        for (int[] row : dist) {
        	// 1 <= grid.length == grid[0].length <= 100, so max distance is 200
            Arrays.fill(row, 201);
        }
        
     	// Top Left to Bottom Right
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (grid[i-1][j-1] == 1) {
                    dist[i][j] = 0;
                } else {
                    dist[i][j] = Math.min(dist[i][j], Math.min(dist[i-1][j]+1, dist[i][j-1]+1));   
                }
            }
        }
        
      	// Bootom Right to Top Left
        for (int i = m; i > 0; i--) {
            for (int j = n; j > 0; j--) {
                dist[i][j] = Math.min(dist[i][j], Math.min(dist[i+1][j]+1, dist[i][j+1]+1));   
            }
        }
       
        int ans = -1;
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (grid[i-1][j-1] == 0) {
                    ans = Math.max(ans, dist[i][j]);
                }
            }
        }
        
        return ans == 201 ? -1 : ans;
    }   
}
```



### 1163. Last Substring in Lexicographical Order

Since the substring are ordered in lexicographical order. After some observation, we can find out two properties that the answer (the last last substring of `s` in lexicographical order) should have:

1. It should always be a suffix of the given string.
2. It should starts with the max letter/character of the given string. 

Lets denote suffix string start with index i as `suffix[i]`. So one thing we can do is that we can find out all the index `i` of the max letter and compare all the corresponding `suffix[i]`. But this will cause TLE when there too many such max letter in the string. So wee need some deduplication strategy to optimize our solution.

After further observation, we found that when there are contiguous letter from index `i` to `j`. 

For index `k` between `i` and `j`,`suffix[k = i] > suffix[i < k <= j]`

For example, in a string `azzzzbc`

```
index 0 1 2 3 4 5 6
char	a z z z z b c

We have contiguous letter z from index 1 to 4
suffix[1] > suffix[2] > suffix[3] > suffix[4] 
zzzzbc    > zzzbc     > zzbc      > zbc
```

So when we have have contiguous duplicates, we only need to check the first index. 

```java
class Solution {    
    public String lastSubstring(String s) {
        int len = s.length();
        char[] chars = s.toCharArray();
        
        // the answer is always a suffix of the given string start with the max character. 
        int maxChar = 0;
        for (int i = 0; i < len; i++) {
            maxChar = Math.max(maxChar, chars[i]);
        }
        
        int startIdx = -1;
        for (int i = 0; i < len; i++) {
            if (chars[i] == maxChar) {
                startIdx = startIdx == -1 ? i : largerSubstring(chars, startIdx, i);
            }
        }
        
        return s.substring(startIdx,len);
    }
    
    private int largerSubstring(char[] chars, int startIdx, int curr) {
        // only need to check the first index as if there is contiguous duplicates. 
        if (chars[curr] != chars[curr-1]) {
            for (int i = 1; curr+i < chars.length; i++) {
                if (chars[startIdx + i] < chars[curr + i]) {
                    return curr;
                } 
                
                if (chars[startIdx + i] > chars[curr + i]) {
                    return startIdx;
                }
            }
        }
        
        return startIdx;
    }
}
```



