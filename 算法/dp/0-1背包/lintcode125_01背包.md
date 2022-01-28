```java
package test.com.fisec.vpn.util;

import java.util.Scanner;


/**
 * 二维数组
 */
class Solution {
    public static int backPackII(int m, int[] A, int[] V) {
        // write your code here
        int n = A.length;
        //这里的dp[i][j]表示从前i个元素中选，体积不超过j的价值的最大值
        //要预留出 当i=0或j=0 。也就是从前0个元素或者背包体积为0的时候
        int dp[][] = new int[n + 1][m + 1];

        for (int i = 1; i <= n; i++) {
            for (int j = 0; j <= m; j++) {
                dp[i][j] = dp[i - 1][j];
                if (j >= A[i - 1]) {
                    //这里要注意第1个元素代表A[0]，因此i对应=>A[i-1],V[i-1]
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - A[i - 1]] + V[i - 1]);
                }
            }
        }
        return dp[n][m];
    }

}
/**
 * 一维数组
 */
class Solution2 {
    public  int backPackII(int m, int[] A, int[] V) {
        // write your code here
        int n = A.length;
        //这里的dp[i][j]表示从前i个元素中选，体积不超过j的价值的最大值
        //要预留出 当i=0或j=0 。也就是从前0个元素或者背包体积为0的时候
        int dp[] = new int[m + 1];
        for (int i = 1; i <= n; i++) {
            for (int j = m; j >= A[i - 1]; j--) {
                //这里要注意第1个元素代表A[0]，因此i对应=>A[i-1],V[i-1]
                dp[j] = Math.max(dp[j], dp[j - A[i - 1]] + V[i - 1]);
            }
        }
        return dp[m];
    }

}

```
