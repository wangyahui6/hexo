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