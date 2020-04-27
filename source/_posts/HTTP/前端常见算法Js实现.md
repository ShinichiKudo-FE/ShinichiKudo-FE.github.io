---
title: 前端常见算法Js实现
date: 2018-08-10 16:28:02
tags: "算法"
categories: "Js"
---

# 快速排序

```js
var arr = [2,53,53.2,1,1223,5,2,675,967,2,1231,9344,3855,1];
```

## 冒泡排序

**原理**：
比较相邻的两个元素，比较大的放置右端

```js
function bubbleSort(arr){
    var i = j =0
    for(var i = 1;i < arr.length;i++){
        for(var j = 0;j <= arr.length - i;j++){
            var temp = 0
            if(arr[j] > arr[j+1]){
                temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
            }
        }
    }
}

// bubbleSort(arr);
// console.log(arr)
```

## 快排

**原理：**
根据基数将分为两部分，一部分的数据要比另一部分的所有数据都要小，然后进行递归调用，直到最后一个元素
<!-- more -->

```js
function quickSort(arr){
    if (arr.length <= 1) { return arr; }

　　var pivotIndex = Math.floor(arr.length / 2) ;

　　var pivot = arr.splice(pivotIndex, 1)[0];

　　var left = [];

　　var right = [];

　　for (var i = 0; i < arr.length; i++){

　　　　if (arr[i] < pivot) {

　　　　　　left.push(arr[i]);

　　　　} else {

　　　　　　right.push(arr[i]);

　　　　}

　　}

   　return quickSort(left).concat([pivot], quickSort(right));

}

// console.log(quickSort(arr));
```

## 插入排序

**原理**
1） 从第一个元素开始，该元素可以认为已经被排序

2） 取出下一个元素，在已经排序的元素序列中从后向前扫描

3） 如果该元素（已排序）大于新元素，将该元素移到下一位置

4） 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置

5）将新元素插入到下一位置中

6） 重复步骤2

```js
function insertSort(arr){
// 假设第0个元素开始是有序排序，第1个元素之后是无序排序
// 所以从第1个元素将无需数列的元素插入有序数列中
for(var i = 1; i < arr.length; i++){
    // 升序
    if(arr[i] < arr[i - 1]){
        // 取出无序数组中弟i个元素作为被插入元素
        var guard = arr[i];
        // 记住有序数列的最后一个元素位置，并将数列位置扩大一个
        var j = i - 1;
        arr[i] = arr[j];
        // 比大小找到被插入元素的位置
        while (j >= 0 && guard < arr[j]) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = guard; // 插入
    }
}
}
insertSort(arr);
console.log(arr)
```

## 二分查找

**原理**
二分查找，也为折半查找。首先要找到一个中间值，通过与中间值比较，大的放又，小的放在左边。
再在两边中寻找中间值，持续以上操作，直到找到所在位置为止。

```js
function binarySort(left,right){
    var result = [],
        il = 0,
        ir = 0;

    while (il < left.length && ir < right.length) {
        if (left[il] < right[ir]) {
            result.push(left[il++]);
        } else {
            result.push(right[ir++]);
        }
    }
    while(left[il]){
        result.push(left[il++]);
    }
    while(right[ir]){
        result.push(right[ir++]);
    }
    return result;
}
console.log(binarySort(arr,1))
```
