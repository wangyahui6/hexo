---
title: javascript排序算法
date: 2018-12-25 11:29:26
description:
categories:
tags:
---
### 冒泡排序

```js
function bubbleSort(arr){
  if(Array.isArray(arr)){
    let length = arr.length;
    for(let i = 0; i < length-1; i++){
      for(let j = 0; j < length - i -1; j++){
        if(arr[j] > arr[j+1]){
          let tmp = arr[j];
          arr[j] = arr[j+1];
          arr[j+1] = tmp;
        }
      }
    }
    return arr;
  }
  return [];
}
```
```js
function bubbleSort2(arr){
  if(Array.isArray(arr)){
    let length = arr.length;
    let pos = length -1;
    while(pos){
      let tempPos = 0;
      for(let j = 0; j < pos; j++){
        if(arr[j] > arr[j+1]){
          tempPos = j;
          let tmp = arr[j];
          arr[j] = arr[j+1];
          arr[j+1] = tmp;
        }
      }
      pos = tempPos;
    }
    return arr;
  }
  return [];
}
```
```js
function bubbleSort3(arr){
  let length = arr.length;
  let low = 0;
  let high = length - 1;
  while(low < high){
    for(let i = low; i < high; i++){
      if(arr[i] > arr[i+1]){
          let tmp = arr[i];
          arr[i] = arr[i+1];
          arr[i+1] = tmp;
        }
    }
    --high;
    for(let j = high; j > low; j--){
      if(arr[j] < arr[j-1]){
          let tmp = arr[j];
          arr[j] = arr[j-1];
          arr[j-1] = tmp;
        }
    }
    ++low;
  }
  return arr;
}
```

#### 选择排序

```js
function selectSort(arr){
  let length = arr.length;
  for(let i = 0; i < length - 1; i++){
    let min = arr[i], 
    minIndex = i;
    for(let j = i + 1; j < length; j++){
      if(min > arr[j]){
        min = arr[j];
        minIndex = j;
      }
    }
    let tmp = arr[i];
    arr[i] = arr[minIndex];
    arr[minIndex] = tmp;
  }
  return arr;
}
```

#### 插入

```js
function insertionSort(arr){
  let length = arr.length;
  for(let i = 1; i < length; i++){
    j = i;
    let temp = arr[j];
    while(j>0 && arr[j-1] > temp){
      arr[j] = arr[j-1];
      j--;
    }
    arr[j] = temp;
  }
  return arr;
}
```

#### 归并排序

```js
function mergeSort(arr){
  let length = arr.length;
  if(length <= 1){
    return arr;
  }
  let mid = Math.floor(length/2);
  let left = arr.slice(0, mid);
  let right = arr.slice(mid);
  return merge(mergeSort(left), mergeSort(right));
}

function merge(left, right){
  let il=0;
  let ir=0;
  let result=[];
  while(il < left.length && ir < right.length){
    if(left[il] < right[ir]){
      result.push(left[il++]);
    }else{
      result.push(right[ir++]);
    }
  }
  while(il < left.length){
    result.push(left[il++]);
  }
  while(ir < right.length){
    result.push(right[ir++]);
  }
  return result;
}

```

#### 快速排序

```js
function quickSort(arr){
  return quick(arr, 0, arr.length - 1);
}
function quick(arr, left, right){
  let length = arr.length;
  if(length > 1){
    let index = partition(arr, left, right);
    if(left < index - 1){
      quick(arr, left, index - 1);
    }
    if(right > index)
    quick(arr, index, right);
  }
  return arr;
}
function partition(arr, left, right){
  let mid = Math.floor((left+right) / 2);
  let tmp = arr[mid];
  while(left <= right){
    while(arr[left] < tmp){
      left++;
    }
    while(arr[right] > tmp){
      right--;
    }
    if(left<=right){
      swap(arr, left, right);
      left++;
      right--;
    }
  }
  return left;
}
function swap(arr, index1, index2){
  let tmp = arr[index1];
  arr[index1] = arr[index2];
  arr[index2]=tmp;
}
```