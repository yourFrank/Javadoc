```java
package test.com.fisec.vpn.util;

import sun.reflect.generics.tree.Tree;

import java.util.ArrayList;
import java.util.Deque;
import java.util.LinkedList;
import java.util.List;

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

    public List<List<Integer>> levelOrderBottom(TreeNode root) {
        LinkedList<List<Integer>> res = new LinkedList<>();
        if (root != null) {
            Deque<TreeNode> deque = new LinkedList();
            deque.add(root);
            while (!deque.isEmpty()) {
                int size = deque.size();
                ArrayList<Integer> list = new ArrayList<>();
                for (int i = 0; i < size; i++) {
                    TreeNode poll = deque.poll();
                    if (poll.left != null) {
                        deque.add(poll.left);
                    }
                    if (poll.right != null) {
                        deque.add(poll.right);
                    }
                    list.add(poll.val);
                }
                res.addFirst(list);
            }
        }
        return res;
    }
}






```
