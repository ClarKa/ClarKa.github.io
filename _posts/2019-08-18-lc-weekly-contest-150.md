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



