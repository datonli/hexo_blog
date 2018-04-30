---
title: 二分查找做旋转数组题
date: 2018-04-29 22:22:27
tags: algorithm
---
最近做了一道简单的题目，回忆了一下二分查找怎么写。

```
题目描述
把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。 输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。 例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。 NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。
```
一开始用最简单的从头到尾遍历一遍，O(n)时间复杂度；不太过瘾，再用二分查找做一遍，时间复杂度O(nlogn)。
```
#include "common.h"

class Solution {
public:
    int binary(vector<int> rotateArray, int left, int right) {
        if (left == right) {
            return rotateArray[left];
        } else if (left + 1 == right) {
            return std::min(rotateArray[left], rotateArray[right]);
        }
        int middle = (right + left)/2;
        int left_min_number = binary(rotateArray, left, middle - 1);
        int right_min_number = binary(rotateArray, middle + 1, right);
        cout << "mid [" << middle << "] = "<< rotateArray[middle] << ", left [" << left << "]= " << left_min_number
            << ", right [" << right << "] = " << right_min_number << endl;
        return std::min(rotateArray[middle], std::min(left_min_number, right_min_number));
    }

    int minNumberInRotateArray(vector<int> rotateArray) {
        if (rotateArray.size() == 0) {
            return 0;
        } else if (rotateArray.size() == 1) {
            return rotateArray[0];
        }
        return binary(rotateArray, 0, rotateArray.size()-1);
    }
};

int main() {
    vector<int> rotateArray{6501,6828,6963,7036,7422,7674,8146,8468,8704,8717,9170,9359,9719,9895,9896,9913,9962,154,293,334,492,1323,1479,1539,1727,1870,1943,2383,2392,2996,3282,3812,3903,4465,4605,4665,4772,4828,5142,5437,5448,5668,5706,5725,6300,6335};
    Solution sol;
    cout << sol.minNumberInRotateArray(rotateArray) << endl;
    return 0;
}
```

很惊喜，然后发现还有一种时间复杂度是O(logn)的解法：
```
    int minNumberInRotateArray(vector<int> rotateArray) {
        if (rotateArray.size() == 0) {
            return 0;
        } else if (rotateArray.size() == 1) {
            return rotateArray[0];
        }
        int right = rotateArray.size() - 1;
        int left = 0;
        int middle = 0;
        while(right > left) {
            middle = (left + right)/2;
            if (rotateArray[middle] > rotateArray[right]) {
                if (middle + 1 == right) {
                    return rotateArray[right];
                }
                left = middle;
            } else if (rotateArray[left] > rotateArray[middle]) {
                if (left + 1 == middle) {
                    return rotateArray[middle];
                }
                right = middle;
            } else {
                return rotateArray[0];
            }
            cout << "mid [" << middle << "] = "<< rotateArray[middle] << ", left [" << left << "]= " << rotateArray[left]
            << ", right [" << right << "] = " << rotateArray[right] << endl;
        }
        return rotateArray[left];
    }
```