```java
package com.fisec.vpn.util;

import java.util.*;



class Solution {
    public List<List<Integer>> permuteUnique(int[] nums) {
        List<List<Integer>> res=new ArrayList<>();
        List<Integer> path=new ArrayList<>();
        //这里数组要排序
        Arrays.sort(nums);
        boolean[] isUsed=new boolean[nums.length];
        dfs(nums,path,res,isUsed);
        return res;
    }

    private void dfs(int[] nums, List<Integer> path, List<List<Integer>> res,boolean[] isUsed) {
        if (path.size()==nums.length){
            res.add(new ArrayList<>(path));
            return;
        }
        for (int i = 0; i < nums.length; i++) {
            if (isUsed[i]){
                continue;
            }
            //用来判重的,i>0保证了i-1不越界
            if (i>0&&nums[i]==nums[i-1]&&!isUsed[i-1]){
                continue;
            }
            isUsed[i]=true;
            path.add(nums[i]);
            dfs(nums,path,res,isUsed);
            path.remove(path.size()-1);
            isUsed[i]=false;
        }
    }

}

```
