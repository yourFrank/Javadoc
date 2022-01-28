```java
package com.fisec.vpn.util;


import java.time.Period;
import java.util.*;

class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;

    TreeNode() {
    }

    TreeNode(int val) {
        this.val = val;
    }

    TreeNode(int val, TreeNode left, TreeNode right) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}

//最开始是对每一个节点进行递归求路径，很多重复的操作
class Solution2 {
    private int sum = 0;

    public int pathSum(TreeNode root, int targetSum) {
        dfs(root, targetSum);
        return sum;
    }

    private void dfs(TreeNode root, int targetSum) {
        if (root == null) {
            return;
        }
        path(root, targetSum);
        dfs(root.left, targetSum);
        dfs(root.right, targetSum);
    }

    private void path(TreeNode root, int targetSum) {
        if (root == null) {
            return;
        }
        if (root.val == targetSum) {
            sum++;
        }
        path(root.left, targetSum - root.val);
        path(root.right, targetSum - root.val);
    }
}

//对于区间的操作，可以使用前缀和来
class Solution {
    public int pathSum(TreeNode root, int targetSum) {
        HashMap<Long, Integer> prefix = new HashMap<>();
        prefix.put(0L, 1);
        return dfs(root, prefix, 0, targetSum);
    }

    public int dfs(TreeNode root, Map<Long, Integer> map, long pre, int targetSum) {
        if (root == null) {
            return 0;
        }

        int ret = 0;
        pre += root.val;

        ret = map.getOrDefault(pre - targetSum, 0);
        map.put(pre, map.getOrDefault(pre, 0) + 1);
        ret += dfs(root.left, map, pre, targetSum);
        ret += dfs(root.right, map, pre, targetSum);
        map.put(pre, map.getOrDefault(pre, 0) - 1); //回溯的时候哈希表中删除当前前缀和

        return ret;
    }
}




```
