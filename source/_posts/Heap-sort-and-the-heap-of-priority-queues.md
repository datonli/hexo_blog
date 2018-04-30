---
title: 堆排序和最大堆的优先队列
date: 2018-05-01 02:42:10
tags: algorithm
---

堆排序新建最差时间复杂度：O(nlogn)

因为是满二叉树，所以树高度：O(logn)+1

最大堆的优先队列，取最大值时间复杂度：O(1)，但是每次取都要调整堆，最差时间复杂度O(logn)

插入一个节点，因为只能插到最后的叶子节点(如果插入到root节点，将root节点送到最后的叶子结点，仅仅调整root进行heapify会造成新叶子结点部分无法处理，还是要整个堆调整)，涉及整个堆的调整，所以最差时间复杂度O(nlogn)；update一个节点的时间复杂度也是O(nlogn)。`这部分据算法导论说可以优化到O(logn)，后面再更新吧`

```
#include "common.h"

class HeapSort {
public:
    HeapSort(vector<int> arr) : arr_(arr.size() * 2, 0), start_(1), end_(arr.size()-1) {
        for (int i = 1; i < arr.size(); ++i) {
            arr_[i] = arr[i];
        }
    }
    virtual ~HeapSort() {}
    void MaxHeapify(vector<int>& arr, int i) {
        if (i < start_) return;
        int left = 2*i;
        int right = 2*i+1;
        int largest = i;
        if (left <= end_ && arr[left] > arr[largest]) {
            largest = left;
        }
        if (right <= end_ && arr[right] > arr[largest]) {
            largest = right;
        }
        if (largest != i) {
            int swap_tmp = arr[i];
            arr[i] = arr[largest];
            arr[largest] = swap_tmp;
            MaxHeapify(arr, largest);
            cout << "swap : arr[" << i << "] = " << arr[i] << ", arr[" << largest << "] << "<< arr[largest] << endl;
        }
        cout << "i = " << i << " ";
        PrintHeap();
    }
    void BuildHeap() {
        for (int i = end_/2; i >= start_; --i) {
            MaxHeapify(arr_, i);
        }
    }
    int RemoveRoot() {
        cout << "Remove Root" << endl;
        int root = arr_[start_];
        arr_[start_] = arr_[end_--];
        //BuildHeap(); // O(nlogn)
        MaxHeapify(arr_, start_); //O(logn)
        PrintHeap();
        return root;
    }

    void InsertValue(int value) {
        cout << "Insert value" << endl;
        arr_[++end_] = value;
        BuildHeap(); // O(nlogn)
        PrintHeap();
    }
    void UpdateValue(int index, int value) {
        if (index < start_ || index > end_) return;
        cout << "Update value : " << arr_[index] << " => " << value << endl;
        arr_[index] = value;
        BuildHeap(); // O(nlogn)
        PrintHeap();
    }
    void PrintHeap() {
        for (int i = start_; i <= end_; ++i) {
            cout << arr_[i] << ", ";
        }
        cout << endl;
    }
private:
    vector<int> arr_;
    int start_;
    int end_;
};

int main() {
    vector<int> arr{0, 8, 1, 16, 14, 7, 9, 4, 10, 3, 2};
    HeapSort hsort(arr);
    hsort.BuildHeap();
    hsort.PrintHeap();
    cout << "remove root = " << hsort.RemoveRoot() << endl;
    hsort.InsertValue(43);
    hsort.UpdateValue(5, 80);
    return 0;
}
```
