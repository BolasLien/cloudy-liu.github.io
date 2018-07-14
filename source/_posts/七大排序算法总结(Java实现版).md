---
title: 七大排序算法总结(Java实现版)
date: 2018-06-14 23:09:33
tags: [Java,排序,数据结构与算法]
---

这节开始总结7种最常用的排序算法，先来看看最简单的冒泡算法。<!-- more --> 

所有完整源代码点击[github 这里](https://github.com/cloudy-liu/DataStructureAndAlgorithm/tree/master/%E4%B8%83%E5%A4%A7%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95) 查看。我们假设测试数据集为以下数组，要求按照从小到达的升序排列。

而排序的动图演示，可以参考 http://www.atool.org/sort.php 

```java
 public static void main(String[] args) {
        int[][] data = new int[][]{
                new int[]{10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0},//逆序
                new int[]{1, 2, 3, 4, 5, 6, 7},//正序
                new int[]{1, 7, 3, 5, 0, 2, 4, 8, 6}};//随机

        for (int i = 0; i < data.length; i++) {
            System.out.println("before sort");
            BaseSort.printArray(data[i]);
            ...
            System.out.println("after sort");
            BaseSort.printArray(data[i]);
        }
    }
```

##  冒泡排序

### 思路

基于相邻比较和交换的方法。针对 n个数，一共会有 n-1 轮。每轮将确定一个值的最终位置，比如升序，那么每轮将会该轮中最大的数的位置确定。

### Java 实现 

先实现交换的辅助函数

```Java
    public void swap(int[] a, int indexI, int indexJ) {
        if (a == null || indexI >= a.length || indexJ >= a.length) {
            System.out.println("a is null ,index out of bound");
            return;
        }
        int tmp = a[indexI];
        a[indexI] = a[indexJ];
        a[indexJ] = tmp;
    }
```

冒泡排序实现

```java
    public void bubbleSort(int[] input) {
        int rightSide = input.length;
        while (rightSide - 1 > 0) { // 控制比较次数，比较次数为 len-1
            for (int i = 1; i < rightSide; i++) { //每轮下两两比较
                if (input[i - 1] > input[i]) {
                    swap(input, i - 1, i);
                }
            }
            --rightSide;
        }
    }
```

你可以看到，当测试集为正序时候，仍然需要$O(n^2)$ 的复杂度，其实每一轮遍历中，两两比较，若整个序列没有发生交换，其实排序就已经可以停止了，比如正序情况下，第一轮两两比较时，由于都是正序，所以一边遍历后没有任何交换，此时就需要停止了。若发生交换，则记录最后一个交换的位置，则该位置为下次交换遍历的终点位置。减少不必要的比较。优化的代码如下

```java
    public void bubbleSortOpt(int[] input) {
        int rightSide = input.length;
        int lastCompareIndex = rightSide;//记录最后一个比较的右边界，同时充当哨兵，是否停止
        while (lastCompareIndex > 0) {//是否要停止
            rightSide = lastCompareIndex;
            lastCompareIndex = 0; //若未发生交换，则会停止
            for (int i = 1; i < rightSide; i++) {
                if (input[i - 1] > input[i]) {
                    swap(input, i - 1, i);
                    lastCompareIndex = i;//记录最后一个发生交换的位置，下次比较的右边界
                }
            }
        }
    }
```

这样复杂度，在最好的情况下（已经有序），可以变为$O(n)$ 

### 复杂度分析

**间复杂度**

排序算法的时间复杂度在于交换和比较的次数，因此交换和比较次数的总和越小，那么时间复杂度越低，冒泡排序一个有 n -1 轮的遍历，每轮遍历中，比较次数依次为 n-1,n-2,...1 。因此比较次数的总和为

$sum=\sum_{1}^{n-1}=1+2+...+n-1=\dfrac{n(n-1)}{2}$ 为 $O(n^2)$

最差情况下（逆序），那么比较一次，就要交换一次。因此总次数为 n(n-1) ，为 $O(n^2)$ ，而最好的情况下，已序的情况下，经过优化后的冒泡为 n-1 次完成，$O(n)$ ,因此平均复杂度为 $O(n^2)$ 。

**空间复杂度**

因为都是固定的几个局部变量，因此空间复杂度为 O(1)

**稳定性**

因为是基于比较，所以冒泡排序是稳定，所谓稳定性就是指相等的两个数 a,b，排序后，a,b的前后位置不变，若a在b前，则排序后仍是a在b前，否则称为不稳定。


## 选择排序

### 思路 

选择排序也是生活中比较常见的一种思路。和冒泡相比，它减少了交换的次数，冒泡相邻的两个数比较，一旦符合交换条件，就会进行交换。选择排序则是一次性找到最后的index值，在一次性的交换。

### Java 实现 

```java
    public void selectSort(int[] data) {//升序排序
        int len = data.length;
        for (int i = 0; i < len; i++) {
            int minIndex = i; //初始化假定i为最小值的索引
            for (int j = i + 1; j < len; j++) {//从 i+1处，开始寻找最小值的索引
                if (data[minIndex] > data[j]) {
                    minIndex = j;
                }
            }
            if (minIndex != i) { //比较一轮下来后，minIndex位置若发生变化，交换
                swap(data, minIndex, i);//swap函数实现间完整代码中
            }
        }
    }
```


### 复杂度分析

**时间复杂度**

由于是逐一确定每个索引位置处最合适的值，因此最好的情况和最差情况，都需要进行比较,

$Sum=\sum_{i=1}^{n-1}=1+2+3+..+n-1=\dfrac{n(n-1)}{2}$，因此复杂度均为为 $O(n^2)$ 

**空间复杂度**

因为都是固定的几个局部变量，因此空间复杂度为 O(1)

**稳定性**

不稳定。举个例子分析，3,1,3, 2 ，第一趟时 index=0为3，逐一确定最小值的索引，所以此时2为最小的值，因此第一个3要与2交换，这样第一个3就在第二个3后面，破换了相等两个值之前的顺序了。因此是不稳定的。

## 直接插入排序

### 思路

直接插入排序基本思想是将一个数插入一个已经有序的数组中，默认有序的为index=0的第一个数，然后逐渐将第i个数插入[0~i-1] 这样有序的数组中。因此可以看出直接插入排序，在已经基本有序的数组中，排序效率是很高的。

### Java实现

```java
    public void insertSort(int[] data) {
        int len = data.length;
        //思想：将一个数插入已经有序的数组中，刚开始有序的为index=0的一个数
        for (int i = 1; i < len; i++) {
            if (data[i] < data[i - 1]) {//若不满足，则该值已经有序,无须插入
                int tmp = data[i]; //保存 data[i]值，后面 data[i]位置要被填坑
                int j;
                for (j = i - 1; j >= 0 && tmp < data[j]; j--) {//逐一向前插入，数组移动
                    data[j + 1] = data[j];
                }
                data[j + 1] = tmp;//j+1位置为需要插入的位置
            }
        }
    }
```

### 复杂度分析

**时间复杂度**

* 最好情况下，每次插入前的比较都不满足，所以遍历一遍就结束 ，复杂度为 $O(n)$

* 最差情况下，从i=1处开始，每次都要发生数组移动，移动次数为 $1+2+3+...+n-1=\dfrac{n(n-1)}{2}$

  比较次数也一样，因此复杂度为 $O(n^2)$   

**空间复杂度** 

几个局部变量，复杂度为 $O(1)$

**稳定性**

基于比较，因此能保持相等的两个数的相对位置，是稳定的。

## 希尔排序

### 思路

借助直接插入排序。基本思想是将整个数组分步逐渐变为基本有序，到全有序。首先确定step，就是将整个数组分组，比如 step=2。那么就是该数组 索引位置 [0,2,4,6,..] 的子序列做插入排序，这样子序列完成有序。接着在step=1.那么就是所有的数组在做插入排序。

### Java实现

```java
     public void shellSort(int[] data) {
        int len = data.length;
        for (int step = len / 2; step > 0; step /= 2) {//将数组分为几次做插入排序
            for (int i = step; i < len; i++) {//step 为间隔
                if (data[i - step] > data[i]) {//和插入排序一样，
                    int tmp = data[i];
                    int j;
                    for (j = i - step; j >= 0 && tmp < data[j]; j -= step) {//只不过 -1换成-step
                        data[j + step] = data[j];
                    }
                    data[j + step] = tmp;
                }
            }
        }
    }
```

### 复杂度分析

**时间复杂度**

希尔排序是第一批突破 $O(n^2)$ 的算法，算法的复杂度跟 step 步长有关系。见[维基百科](https://zh.wikipedia.org/wiki/%E5%B8%8C%E5%B0%94%E6%8E%92%E5%BA%8F)


**空间复杂度** 

几个局部变量，复杂度为 $O(1)$

**稳定性**

由于是跳跃式的移动，因此是不稳定的。

## 归并排序

### 思路

归并的意思包含两层，一个是分治，一个是合并。思想是将一个待排序的数组，分为两个有序的数组，然后将这两个有序数组合并为一个。

### Java实现 

```java
    public void mergeSort(int[] data) {
        int len = data.length;
        int[] tmp = new int[len];
        mergeSortImplementRecur(data, 0, len - 1, tmp);
    }

    //递归实现
    private void mergeSortImplementRecur(int[] data, int firstIndex, int lastIndex, int[] tmp) {
        if (firstIndex >= lastIndex) {//递归的截止条件
            return;
        }
        int midIndex = (firstIndex + lastIndex) / 2;
        mergeSortImplementRecur(data, firstIndex, midIndex, tmp);
        mergeSortImplementRecur(data, midIndex + 1, lastIndex, tmp);
        mergeArray(data, firstIndex, midIndex, lastIndex, tmp);
    }

   //非递归实现，TODO

    private void mergeArray(int[] data, int firstIndex, int midIndex, int lastIndex, int[] tmp) {
        int i = firstIndex;
        int n = midIndex;
        int j = midIndex + 1;
        int m = lastIndex;
        int k = 0;
        while (i <= n && j <= m) {//逐一比较，将小的放入到 tmp数组中
            if (data[i] < data[j]) {
                tmp[k++] = data[i++];
            } else {
                tmp[k++] = data[j++];
            }
        }
        while (i <= n) {//两个数组长度不一致时
            tmp[k++] = data[i++];
        }
        while (j <= m) {
            tmp[k++] = data[j++];
        }
        for (i = 0; i < k; i++) {//复制到tmp数组中
            data[firstIndex + i] = tmp[i];
        }
    }
```

###  复杂度分析

**时间复杂度**

每次合并将两个有序的数组合并一起，需要遍历一次，因此复杂度为$O(n)$ ，而要合并多少次？由于二分，所以次数为 $log_2n$ 次，所以总的时间复杂度： $nlog_2n$ 。进一步可以到，不管是正序，逆序复杂度均一样，因此效率比较高。

**空间复杂度**

因为归并排序使用了一个同等规模的额外辅助数组，这个复杂度为 $O(n)$，采用递归调用的话，因为递归栈的深度为 $log_2n$，因此总的是$O(n+log_2n)$。若采用的是非递归，那么递归栈部分可以省略，效率更高，复杂度为 $O（n)$ 。因此优先使用的是非递归的实现。

**稳定性**

由于也是基本两两比较，因此也是稳定性的

## 堆排序

TODO

## 快速排序

### 思路

“分治”+"填坑“法。基本思路为，选取一个基准值，比如index=0的值，然后唯一确定这个基准的位置，使得该值前面的数都小于它，后面的数都大于它，这样就唯一确定了这个基准值的位置。然后在分别进行递归操作，逐一逼近单一数组值。而在确定基准值时候，是采用两头法，假设基准值为index=0的值，先保存好基准值，然后先从右游标向左找起，若是升序排序，则一直找到比基准值小的位置，停住。然后将该值填入基准值的坑，然后在从左往右找大于基准值的位置，找到后，将该值插入右边出现的坑。直到左右游标相遇，则该值就是基准值的位置填入，这样一轮下来，就唯一确定基准值了。然后在递归左序列和右序列。

### Java 实现

```java
     public void quickSort(int[] data, int start, int end) {
        if (start >= end) {
            return;
        }
        int base = data[start];//start 位置为基准值
        int i = start;
        int j = end;

        while (i < j) {
            while (i < j && data[j] >= base) {//从右往左找小于的位置 j
                j--;
            }
            if (i < j) {
                data[i++] = data[j];
            }
            while (i < j && data[i] <= base) { //从左往右找大于基准值的i
                i++;
            }
            if (i < j) {
                data[j--] = data[i];
            }
        }
        if (i == j) {//此时i为基准值的索引
            data[i] = base;
        }
        quickSort(data, start, i - 1);//递归调用左序列
        quickSort(data, i + 1, end);// 递归调用右序列
    }
```

上面的实现是快排的基本实现，但是可以看出，有许多地方可以进行优化。

* 基准值改进

  首先基准值的选择比较重要，假设基准值每次都是选取了最大或是最小，其实一轮下来，数据量只是减少了一个，假设每次基准值都是选择比较平均，都是中间的值，那么每轮下来，数据量都会减小一半。可以采用取首位，中间值三者的中值来作为基准值。

* 尾递归改进

  由于快排是基于递归实现，所以数据量大的时候，会造成调用栈深度过大，因此改为尾递归调用

* 小数量时，退化为直接插入排序

所以优化后的快排为

```java
public void quickSortOpt(int[] data, int start, int end) {
        if (end - start < MAX_SORT_NUMBER) {//优化1，小数据采用 直接插入法
            insertSort(data, start, end);
        } else {
            while (start < end) {//优化2,尾递归减少调用栈
                int i = start;
                int j = end;

                //优化3。采用中值法，选择基准值
                int mid = (start + end) / 2;
                if (data[start] > data[end]) {
                    swap(data, start, end);
                }
                if (data[mid] > data[end]) {
                    swap(data, mid, end);
                }
                if (data[mid] > data[start]) {
                    swap(data, mid, start);
                }
                int base = data[start];

                while (i < j) {
                    while (i < j && data[j] >= base) {
                        j--;
                    }
                    if (i < j) {
                        data[i++] = data[j];
                    }
                    while (i < j && data[i] <= base) {
                        i++;
                    }
                    if (i < j) {
                        data[j--] = data[i];
                    }
                }
                if (i == j) {
                    data[i] = base;
                }
                quickSort(data, start, i - 1);
                start = i + 1;//优化2 尾递归
            }
        }
    }

    private void insertSort(int[] data, int start, int end) {
        for (int i = start + 1; i <= end; i++) {
            if (data[i - 1] > data[i]) {
                int tmp = data[i];
                int j;
                for (j = i - 1; j >= 0 && tmp < data[j]; j--) {
                    data[j + 1] = data[j];
                }
                data[j + 1] = tmp;
            }
        }
    }
```

### 复杂度分析

从实现来看，快排每次都需要遍历一次数组，$O(n)$ ，而最差的情况，每次分治后，都只减少一个数据，所以一共会有 n-1 次，所以此种情况下，时间复杂度为 $O(n^2)$ ，最好的情况下，是每次分治都减少一半的数据量，因此次数为 $O(logn)$ ，所以最好的情况是 $O(nlogn)$ ，空间复杂度为由于是递归调用，所以$O(logn)$ 

同时又由于是跳跃的比较，所以是不稳定的。

## 总结

|              | 时间复杂度（最好） | 时间复杂度（最差） | 时间平均复杂度 | 空间复杂度 | 稳定性 |
| ------------ | ------------------ | ------------------ | -------------- | ---------- | ------ |
| 冒泡排序     | $O(n)$             | O(n^2)             | $O(n^2)$       | O(1)       | 稳定   |
| 选择排序     | $O(n^2)$           | $O(n^2)$           | $O(n^2)$       | O(1)       | 不稳定 |
| 直接插入排序 | $O(n)$             | $O(n^2)$           | $O(n^2)$       | O(1)       | 稳定   |
| 希尔插入排序 | $O(nlogn)$         | 依据step           | 依据step       | O(1)       | 不稳定 |
| 归并排序     | $O(nlog_2n)$       | $O(nlog_2n)$       | $O(nlog_2n)$   | O(n)       | 稳定   |
| 快速排序     | $O(nlogn)$         | $O(n^2)$           | $O(nlogn)$     | $O(logn)$  | 不稳定 |

# Refs

* [sort_wiki](https://en.wikipedia.org/wiki/Sorting_algorithm#Comparison_of_algorithms)