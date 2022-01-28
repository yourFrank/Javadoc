```java
package com.fisec.vpn.util;

import java.util.*;

class Solution {
    public List<List<Integer>> combinationSum3(int k, int n) {
        List<List<Integer>> res=new ArrayList<>();
        LinkedList<Integer> path = new LinkedList<>();
        dfs(res,path,1,n,k,0);
        return res;
    }

    private void dfs( List<List<Integer>> res,LinkedList<Integer> path, int begin,int n,int k,int sum) {

        if (path.size()==k){
            if (sum==n){
                res.add(new ArrayList<>(path));
            }
            return;
        }

        for (int i = begin; i <= 9; i++) {
            if (sum+i>n){
                break;
            }
            //如果前面一个元素重复，并且已经被用过了
            path.addLast(i);
            dfs(res,path,i+1,n,k,sum+i);
            path.removeLast();
        }
    }
}


```
