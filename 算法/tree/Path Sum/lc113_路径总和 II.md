```java
package test.com.fisec.vpn.util;


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


class Solution {
    List<List<Integer>> res=new ArrayList<>();
    LinkedList<Integer> subList=new LinkedList<>();
    public List<List<Integer>> pathSum(TreeNode root, int targetSum) {

        dfs(root,targetSum);
        return res;
    }

    public void dfs(TreeNode root, int targetSum) {
        if (root==null){
            return;
        }
        subList.add(root.val);
        if (root.left==null&&root.right==null&&targetSum==root.val){
                res.add(new LinkedList<>(subList));
        }

        dfs(root.left,targetSum-root.val);
        dfs(root.right,targetSum- root.val);
        subList.removeLast();

    }
}



```
