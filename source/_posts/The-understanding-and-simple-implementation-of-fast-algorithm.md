---
title: 快排算法的理解和简单实现
date: 2018-05-01 02:44:12
tags: algorithm
---

快排，最经典、最常用的排序算法。平均时间复杂度是O(nlogn)，虽然最差的时间复杂度是O(n^2)，但是很难出现，尤其是随机打乱x的时候。

理解快排重点是两个指针：i和j。**i指针的定位是<=i的都是比x小的；i+1到j-1这个范围都是比x要大的；j指向的是现在正要判断的。**

```
#include "common.h"

class QuickSort {
public:
    static void QuickSortRun(vector<int>& arr, int left, int right) {
        if (left < right) {
            int pov = Partition(arr, left, right);
            QuickSortRun(arr, left, pov-1);
            QuickSortRun(arr, pov+1, right);
        }
    }
private:
    static void SwapNum(vector<int>& arr, int left, int right) {
        int swap_num = arr[left];
        arr[left] = arr[right];
        arr[right] = swap_num;
    }
    static int Partition(vector<int>& arr, int left, int right) {
        int x = arr[right];
        int i = left - 1;
        int j = left;
        for (; j < right; ++j) {
            if (arr[j] < x) {
                i++;
                SwapNum(arr, i, j);
            }
        }
        SwapNum(arr, ++i, right);
        return i;
    }
};

int main() {
    vector<int> arr{0, 8, 1, 16, 14, 7, 9, 4, 10, 3, 2};
    QuickSort::QuickSortRun(arr, 0, arr.size()-1);
    for (auto& a : arr) {
        cout << a << ", ";
    }
    cout << endl;
    return 0;
}
```

要实现随机化版本很简单，就是在left和right之间随机出一个i，把i和right交换下，再调用Partition就完了。