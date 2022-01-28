```java
package com.fisec.vpn.util;

//和旋转数组1的区别在于有相等的元素，此时如果Mid=right相等的时候没法判断取舍区间，要r--
class Solution2 {
    public int findMin(int[] nums) {
        int l = 0;
        int r = nums.length - 1;
        while (l < r) {

            int mid = l + (r - l) / 2;
            if (nums[mid] < nums[r]) { //右侧是单调区间
                r = mid; //保留当前元素
            } else if (nums[mid] > nums[r]) {
                l = mid + 1;
            } else {
                r--;
            }
        }
        return nums[l];
    }

}

//分治，左边的最小值，和右边最小值取min（同153旋转数组）
class Solution {
    public int findMin(int[] nums) {
        return findMin(nums, 0, nums.length - 1);
    }

    private int findMin(int[] nums, int l, int r) {
        //如果剩下一个或者两个元素，选取其中的最小值
        if (l + 1 >= r) return Math.min(nums[l], nums[r]);
        // Sorted,则返回开始的元素l即可
        if (nums[l] < nums[r]) return nums[l];
        int mid = (r - l >> 1) + l;
        return Math.min(findMin(nums, l, mid), findMin(nums, mid + 1, r));
    }
}

```
