# 算法学习之排序算法对比


本文是学习排序系列的第五篇，主要对比三种基本排序算法以及三种进阶排序算法，对应的排序算法学习笔记可以翻阅本博客前面的内容。

<!--more-->

## 0. 常见排序算法性能对比

再贴一张常见排序算法的性能对比，方便查看~

![](https://raw.githubusercontent.com/shmilywh/PicturesForBlog/master/2021/05/26-18-44-47-2021-05-26-18-44-43-image.png)

---

## 1. 代码

```python
import numpy as np
from time import process_time
from typing import List
import matplotlib.pyplot as plt
import random


class Sorting:
    def __init__(self, method: str):
        self.easy_samples = np.random.randint(0, 100000, 5000)
        self.medium_samples = np.random.randint(0, 100000, 50000)
        self.hard_samples = np.random.randint(0, 100000, 100000)
        self.start_time = process_time()
        self.method = getattr(self, method)

    def timeit(self):
        return float('%.4f' % (process_time() - self.start_time))

    def sort(self):
        # print('---------------- Easy Test -------------------')
        easy_nums = self.method(self.easy_samples)
        t1 = self.timeit()
        # print('---------------- Medium Test -------------------')
        medium_nums = self.method(self.medium_samples)
        t2 = self.timeit()
        # print('---------------- Hard Test -------------------')
        hard_nums = self.method(self.hard_samples)
        t3 = self.timeit()
        assert self.checksort(easy_nums)
        assert self.checksort(medium_nums)
        assert self.checksort(hard_nums)
        return [t1, t2, t3], [self.easy_samples.shape, self.medium_samples.shape, self.hard_samples.shape]

    @staticmethod
    def checksort(nums):
        for i in range(len(nums) - 1):
            if nums[i] > nums[i + 1]:
                return False
        return True

    @staticmethod
    def bubblesort(nums: List) -> List:
        length = len(nums)
        for i in range(1, length):
            sort_over = True
            for j in range(length - i):
                if nums[j] > nums[j + 1]:
                    nums[j], nums[j + 1] = nums[j + 1], nums[j]
                    sort_over = False
            if sort_over:
                return nums
        return nums

    @staticmethod
    def selectsort(nums: List) -> List:
        length = len(nums)
        for i in range(length):
            min_id = i
            for j in range(i, length):
                min_id = j if nums[j] < nums[min_id] else min_id
            if min_id != i:    # 判断是否是当前元素最小，是的话就不用交换
                nums[min_id], nums[i] = nums[i], nums[min_id]
        return nums

    @staticmethod
    def insertsort(nums: List) -> List:
        length = len(nums)
        for i in range(length):
            cur_val = nums[i]
            last_id = i - 1

            while last_id >= 0 and nums[last_id] > cur_val:
                nums[last_id + 1] = nums[last_id]
                last_id -= 1

            nums[last_id + 1] = cur_val
        return nums

    @staticmethod
    def shellsort(nums: List) -> List:
        length = len(nums)
        gap = length

        while gap > 0:
            for i in range(gap, length):
                cur_val = nums[i]
                last_id = i - gap
                while last_id >= 0 and nums[last_id] > cur_val:
                    nums[last_id + gap] = nums[last_id]
                    last_id -= gap
                nums[last_id + gap] = cur_val
            gap //= 2

        return nums

    @staticmethod
    def mergesort(nums: List) -> List:
        def merge(arr_, left_, mid_, right_, tmp_):
            ptr1, ptr2, index = left_, mid_+1, 0

            # 遍历ptr1和ptr2，装填res_数组，直到ptr1或ptr2到头
            for i in range(right_-left_+1):
                if ptr1 > mid_ or ptr2 > right_:
                    break
                if arr_[ptr1] <= arr_[ptr2]:   # 注意 '=' 才能让排序稳定
                    tmp_[index] = arr_[ptr1]
                    ptr1, index = ptr1+1, index+1
                else:
                    tmp_[index] = arr_[ptr2]
                    ptr2, index = ptr2+1, index+1
            # 调用extend将剩余元素都装入数组res中
            if ptr1 > mid_:
                tmp_[index:right_-left_+1] = arr_[ptr2:right_+1]
            if ptr2 > right_:
                tmp_[index:right_-left_+1] = arr_[ptr1:mid_+1]
            # 改变arr_区间中的元素顺序
            arr_[left_:right_+1] = tmp_[:right_-left_+1]

        def mergesort_rec(arr, left, right, tmp):
            if left >= right:
                return
            mid = (right + left) // 2
            mergesort_rec(arr, left, mid, tmp)
            mergesort_rec(arr, mid+1, right, tmp)
            merge(arr, left, mid, right, tmp)

        length = len(nums)
        mergesort_rec(nums, 0, length-1, [0]*length)

        return nums

    @staticmethod
    def quicksort(nums: List) -> List:
        def partition(left, right):
            i, pivot = left, nums[left]    # 这里记录下左指针指向的值，后面比较时不需要重复索引，可以有效节省时间
            while left < right:
                while left < right and nums[right] >= pivot:
                    right -= 1
                while left < right and nums[left] <= pivot:
                    left += 1
                nums[left], nums[right] = nums[right], nums[left]
            nums[i], nums[left] = nums[left], pivot
            return left

        def sort(start, end):
            if start >= end:
                return []
            pivot = partition(start, end)
            sort(start, pivot-1)
            sort(pivot+1, end)

        sort(0, len(nums)-1)
        return nums


def plot_and_show(sorting_algo, res, size):
    x = size  # 点的横坐标

    def randomcolor():
        colorArr = ['1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F']
        color = ""
        for i in range(6):
            color += colorArr[random.randint(0, 14)]
        return "#" + color

    for i, cur in enumerate(res):
        print(cur)
        plt.plot(x, cur, 's-', color=randomcolor(), label=sorting_algo[i])  # s-:方形

    plt.xlabel("input scale")  # 横坐标名字
    plt.ylabel("time cost(s)")  # 纵坐标名字
    plt.legend(loc="best")  # 图例
    plt.show()


if __name__ == '__main__':
    sorting_algo = [
                    'bubblesort',
                    'selectsort',
                    'insertsort',
                    'shellsort',
                    'mergesort',
                    'quicksort',
                    ]
    res = []
    size = []
    for cur_algo in sorting_algo:
        sort_ = Sorting(method=cur_algo)
        t, size = sort_.sort()
        res.append(t)

    plot_and_show(sorting_algo, res, size)
```

## 2. 算法运行时间对比

**横轴为数据量级，纵轴为运行时间**

六种算法共同对比

![all.png](https://raw.githubusercontent.com/shmilywh/PicturesForBlog/master/2021/05/28-00-52-55-all.png)

希尔、归并、快排详细对比

![3l.png](https://raw.githubusercontent.com/shmilywh/PicturesForBlog/master/2021/05/28-00-53-13-3l.png)

