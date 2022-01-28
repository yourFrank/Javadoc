```java
package com.fisec.vpn.util;

import java.util.*;
import java.util.stream.Collectors;


class Solution {
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        List<List<Integer>> res = new ArrayList<>();
        //使用boolean[]记录前一个数是否已经操作了
        boolean[] flag = new boolean[candidates.length];
        //让数组有序，方便后面的去重和剪枝
        Arrays.sort(candidates);
        dfs(res, new LinkedList<>(), candidates, target, 0, 0, flag);
        return res;
    }

    public void dfs(List<List<Integer>> res, LinkedList<Integer> path, int[] candidates, int target, int sum, int begin, boolean[] flag) {
        // 找到了数字和为 target 的组合
        if (sum == target) {
            res.add(new ArrayList<>(path));
            return;
        } else if (sum > target) {
            return;
        }
        if (begin == candidates.length) {
            return;
        }
        for (int i = begin; i < candidates.length; i++) {
            //避免重复选择,这里如果是和之前重复的元素，并且已经处理过了（处理过回溯后会变成false, 而!false就是为true）
            if (i > 0 && candidates[i] == candidates[i - 1] && !flag[i - 1]) {
                continue;
            }
            //剪枝操作
            if (sum+candidates[i]>target){
                break;
            }
            flag[i] = true;
            path.addLast(candidates[i]);
            dfs(res, path, candidates, target, sum + candidates[i], i + 1, flag);
            path.removeLast(); // 回溯，移除路径 path 最后一个元素
            flag[i] = false;
        }
    }
}


```
