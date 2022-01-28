```java
package test.com.fisec.vpn.util;

import java.util.Scanner;


/**
 * 二维数组去k优化版
 */
class Solution2 {
    /**
     * @param A: an integer array
     * @param V: an integer array
     * @param m: An integer
     * @return: an array
     */
    public int backPackIII(int[] A, int[] V, int m) {
        // write your code here
        int n=A.length;
        int dp[][]=new int[n+1][m+1];

        for (int i=1;i<=n;i++){
            for (int j=0;j<=m;j++){
                dp[i][j]=dp[i-1][j];
                if (j>=A[i-1]){
            //0-1背包这里是dp[i-1][j-A[i-1]]+V[i-1].区别就在这。具体推导过程看我的博客
                    dp[i][j]=Math.max(dp[i][j],dp[i][j-A[i-1]]+V[i-1]);
                }
            }
        }
        return dp[n][m];
    }
}

/**
 * lintcode 440 完全背包一维数组
 */
 class Solution {
    /**
     * @param A: an integer array
     * @param V: an integer array
     * @param m: An integer
     * @return: an array
     */
    public int backPackIII(int[] A, int[] V, int m) {
        // write your code here
        int n=A.length;
        int dp[]=new int[m+1];

        for (int i=1;i<=n;i++){
            for (int j=A[i-1];j<=m;j++){
                    dp[j]=Math.max(dp[j],dp[j-A[i-1]]+V[i-1]);
            }
        }
        return dp[m];
    }
}

```
