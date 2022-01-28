```java
package com.fisec.vpn.util;

import java.util.*;

class Solution {
    public List<List<Integer>> subsetsWithDup(int[] nums) {
        List<List<Integer>> res=new ArrayList<>();
        LinkedList<Integer> path=new LinkedList<>();
        Arrays.sort(nums);
        Boolean[] isUsed=new Boolean[nums.length];
        dfs(nums,res,path,0,isUsed);
        return res;
    }

    private void dfs(int[] nums, List<List<Integer>> res,LinkedList<Integer> path, int begin,Boolean[] isUsed) {

        res.add(new ArrayList<>(path));

        for (int i = begin; i < nums.length; i++) {
            //如果前面一个元素重复，并且已经被用过了
            if (i>0&&nums[i]==nums[i-1]&&!isUsed[i-1]){
                continue;
            }
            path.addLast(nums[i]);
            isUsed[i]=true;
            dfs(nums,res,path,i+1,isUsed);
            path.removeLast();
            isUsed[i]=false;
        }
    }
}


```
