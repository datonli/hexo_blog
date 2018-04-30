---
title: 查找最长子数组的方法
date: 2018-04-29 22:22:54
tags: algorithm
---
查找最长子数组的方法,是算法导论里一道很有趣的题目。

比如，给定一个数组{13, -3, -25, 20, -3, -16, -23, 18, 20, -7, 12, -5, -22, 15, -4, 7}，找出这里面最长的子数组是什么？

因为最长子数组必定是：
1）处于mid左边；
2）处于mid右边；
3）跨过mid
三种情况。

```

#include "common.h"

class Solution {
public:

    int CrossMidSubArray(vector<int> arr, int mid, int left, int right) {
        int left_sum = -10000;
        int left_pos = -1;
        int pos = mid-1;
        int sum = 0;
        for (; pos >= left; --pos) {
            sum += arr[pos];
            if (sum > left_sum) {
                left_sum = sum;
                left_pos = pos;
            }
        }
        pos = mid + 1;
        int right_sum = -10000;
        int right_pos = -1;
        sum = 0;
        for (; pos <= right; ++pos) {
            sum += arr[pos];
            if (sum > right_sum) {
                right_sum = sum;
                right_pos = pos;
            }
        }
        sum = left_sum + arr[mid] + right_sum;
        cout << " arr[" << mid << "] : ( " << left_pos << ", " << right_pos << "), sum = " << sum << endl; 
        return sum;
    }

    int FindMaxSubArray(vector<int> arr, int left, int right) {
        if (left == right) return arr[right];
        if (left > right) return arr[right];
        if (right < left) return arr[left];
        int mid = (left + right) / 2;
        int left_sum = FindMaxSubArray(arr, left, mid-1);
        int right_sum = FindMaxSubArray(arr, mid+1, right);
        int mid_sum = CrossMidSubArray(arr, mid, left, right);
        return max(left_sum + arr[mid], max(right_sum + arr[mid], mid_sum));
    }

    int LongestContinueArray(vector<int> arr) {
        return FindMaxSubArray(arr, 0, arr.size()-1);
    }
};

int main() {
    vector<int> arr{13, -3, -25, 20, -3, -16, -23, 18, 20, -7, 12, -5, -22, 15, -4, 7};
    Solution sol;
    cout << sol.LongestContinueArray(arr) << endl;
    return 0;
}
```

算法导论最后给出了另一种思路（习题：4.1-5），如果知道arr[1,j]的最大子数组，那么arr[1,j+1]的最大子数组有两种情况：
1）即：arr[1,j]的最大子数组；
2）或：arr[i,j+1]的最长连续子数组。
