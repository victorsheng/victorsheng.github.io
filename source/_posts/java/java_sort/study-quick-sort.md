---
title: study_quick_sort
tags:
  - 排序
categories:
  - java
abbrlink: 440599423
date: 2018-01-22 22:39:55
---
![upload successful](/images/pasted-41.png)

视频地址:
https://visualgo.net/en/sorting

代码:


```
public class QuickSort {


    public static void main(String[] args) throws Exception {
        int[] ints = {5, 3, 9, 1, 6, 7, 2, 4, 0, 8};
        int[] result = new QuickSort().sort(ints);
        System.out.println(Arrays.toString(result));
    }


    public int[] sort(int[] sourceArray) throws Exception {
        // 对 arr 进行拷贝，不改变参数内容
        int[] arr = Arrays.copyOf(sourceArray, sourceArray.length);

        return quickSort(arr, 0, arr.length - 1);
    }

    private int[] quickSort(int[] arr, int left, int right) {
        if (left < right) {
            int partitionIndex = partition(arr, left, right);
            System.out.println(partitionIndex);
            quickSort(arr, left, partitionIndex - 1);
            quickSort(arr, partitionIndex + 1, right);
        }
        return arr;
    }

    private int partition(int[] arr, int left, int right) {
        // 设定基准值（pivot）
        int pivot = left;
        int index = pivot + 1;
        for (int i = index; i <= right; i++) {
            if (arr[i] < arr[pivot]) {
                swap(arr, i, index);
                index++;
            }
        }
        swap(arr, pivot, index - 1);
        return index - 1;
    }

    private void swap(int[] arr, int i, int j) {
        if (i == j) {
            return;
        }
        System.out.println("swap" + i + "----" + j);
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}

```