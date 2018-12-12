---
layout: drafts
title: Promise解析和实现
description: Promise因为它的调用方式使得异步操作清晰简单，是现在异步操作的主要方式。Promise的使用和实现是面试中的高频问点。这骗文章主要解析Promise规范和我自己的一版实现方式。
date: 2018-12-10 17:44:11
tags: js 前端
categories: 前端
---

### PromiseA+规范

了解Promise首先我们要清楚Promise规范的内容，规范规定了Promise的行为和调用方式。[这里是规范原文](https://promisesaplus.com/)。下面是加上自己理解翻译总结：

一个Promise主要代表了一个异步操作的最终结果。与Promise交互的主要方式是通过promise实例的`then`方法注册回调函数。回调函数分为两种，一种接受异步操作成功时的回调，接受异步操作返回值为参数；第二种是异步操作失败时的回调，接受异步操作失败原因为操作。规范主要是详细描述了`then`方法的实现细节。

主要要求如下：

#### 状态
 一个promise必须处于这3中状态之中：`pending`、`fullfilled`、`rejected`
  - `pending`: 可以转换到`fullfilled`或者`rejected`状态
  - `fullfilled`: 不能转换到任何状态且有一个不可变的结果值
  - `rejected`: 不能转换到任何状态且有一个不可变的失败原因


#### `then`方法

一个`promise`必须有一个`then`方法来接受异步操作的结果值或者失败原因

一个`promise`的then应该接收两个参数，成功回调函数和失败回调函数：

```js
promise.then(onFulfilled, onRejected)
```

  - onFulfilled和onRejected都是可选的：
    - 如果onFulfilled不是一个函数，它必须被忽略
    - 如果onRejected不是一个函数，它必须被忽略

  - 如果onFulfilled是函数：
    - 它必须在`promise`状态变为`fullfilled`之后执行，且接受`promise`的结果值作为第一个参数
    - 它在`promise`状态变为`fullfilled`之前不能执行
    - 它最多被执行一次

  - 如果onRejected是函数：
    - 它必须在`promise`状态变为`rejected`之后执行，且接受`promise`的失败原因作为第一个参数
    - 它在`promise`状态变为`rejected`之前不能执行
    - 它最多被执行一次

  - `onFulfilled`或者`onRejected`直到执行环境堆栈只包含平台代码时才可以执行

  - `onFulfilled`或者`onRejected`必须作为一个独立的函数被调用。（不能作为对象属性执行或者其他指定this的执行方式，比如call和apply）

  - `then`方法可以在同一个`promise`上多次调用：
    - 当`promise`状态变为`fullfilled`时，通过then注册的所有`onFulfilled`回调按照注册顺序依次执行
    - 当`promise`状态变为`rejected`时，通过then注册的所有`onRejected`回调按照注册顺序依次执行

  - `then`方法必须返回一个promise：
    ```js
      promise2 = promise1.then(onFulfilled, onRejected);
    ```
      - 如果`onFulfilled`或者`onRejected`返回`x`, 则运行Pomise Resolution Procedure`[[Resolve]](promise2, x)`
      - 如果`onFulfilled`或者`onRejected`抛出一个异常`e`,`promise2`必须以`e`作为失败原因变成`rejected`状态
      - 如果`onFulfilled`不是一个函数，且`promise1`为`fullfilled`状态，则`promise2`也转变为`fullfilled`,且结果值和`promise1`一样
      - 如果`onRejected`不是一个函数，且`promise1`为`rejected`状态，则`promise2`也转变为`rejected`,且失败原因和`promise1`一样

#### The Promise Resolution Procedure

  `promise resolution procedure`表示为`[[Resolve]](promise, x)`,是一个接受一个promise和一个值作为参数的抽象操作。如果一个函数是一个`thenable`（`thenable`表示有`then`方法的对象或者函数），则它试图使`promise`采用`x`的状态。这里对于`thenable`使Promise更加通用，能够兼容之前并不符合规范但是有合理`then`方法的异步实现。

  为了运行`[[Resolve]](promise, x)`，需要执行以下步骤：

  - 如果`promise`和`x`指向同一个对象，则以`TypeError`为失败原因将`promise`转变为`rejected`状态
  - 如果`x`是一个promise，则`promise`采用`x`的状态以及相应的结果值或者原因
  - 如果`x`是一个对象或者函数
    - `Let`定义`then`,值为`x.then`
    - 如果在获取`x.then`的过程中抛出异常`e`,`promise`以`e`作为失败原因变成`rejected`状态
    - 如果`then`是一个函数，以`x`作为它的`this`调用它，第一个参数为`resolvePromise`，第二个参数为`rejectPromise`:
      - 如果`resolvePromise`以参数`y`被调用的话，则执行`[[Resolve]](promise, y)`
      - 如果`rejectPromise`以参数`r`被调用的话,则以`r`作为失败原因使``变成`rejected`状态
      - 如果`resolvePromise`和`rejectPromise`都被调用了，或者以同样的参数被调用多次，则以第一次调用为准，忽略之后的调用
      - 如果调用`then`时抛出一个异常`e`:
        - 如果`resolvePromise`或`rejectPromise`被调用了，则忽略
        - 否则，`promise`以`e`作为失败原因变成`rejected`状态
    - 如果`then`不是一个函数，则`promise`以`x`为结果值变为`fullfilled`状态
  - 如果`x`不是一个对象或者函数，则`promise`以`x`为结果值变为`fullfilled`状态

### Promise实现

我们可以看出标准还是挺复杂的，我们就一步一步，从简到繁地实现Promise。标准中并没有规定Promise如何创建，如何完成状态转换，这些我们可以参考ES6的promise用法：

```js
const promise1 = new Promise((resolve, reject) => {

});
promise1.then((value) => {}, (reason)=>{})
```
#### promise的状态
Promise以类的方式出现，构造函数接受一个函数`fn`作为参数，`fn`有两个参数：

- `resolve`,使promise从`pending`状态转换为`fullfilled`状态，`resolve(value)`的参数value为promise的结果值；
- `reject`,使promise从`pending`状态转换为`rejected`状态，`reject(reason)`的参数reason为promise的失败原因。

我们第一版可以先从状态写起：

```js
const PENDING = 'pending';
const FULLFILLED = 'fullfilled';
const REJECTED = 'rejected';
class Promise {
  constructor(fn) {
    this.status = PENDING;
    this.value = undefined;
    this.reason = undefined;
    const resolve = (value) => {
      if(this.status === PENDING){
        this.status = FULLFILLED;
        this.value = value;
      }
    }
    const reject = (reason) => {
      if(this.status === PENDING){
        this.status = REJECTED;
        this.reason = reason;
      }
    }
    try{
      fn(resolve, reject);
    }catch(e){
      reject(e);
    }
  }
}
```

try...catch...是为了捕获用户自定义函数`fn`的错误，如果`fn`执行出错，则promise会进入`rejected`状态。

#### then函数的两个函数参数的执行

接下来我们考虑`then`函数，`then`函数比较复杂，我们先考虑`then(onFulfilled, onRejected)`函数中两个函数参数的执行问题，把`then`函数的返回值问题放在后面。

`then`函数中两个函数参数的执行情况分为3种：

- promise处于`pending`状态，那在then函数中不执行两个函数参数，等到`fn`中的异步操作有结果了，再根据成功或者失败的结果去执行

- promise处于`fullfilled`状态，则直接执行`then`函数的第一个函数参数`onFulfilled`

- promise处于`rejected`状态，则直接执行`then`函数的第二个函数参数`onRejected`

```js
class Promise {
  constructor(fn) {
    this.status = PENDING;
    this.value = undefined;
    this.reason = undefined;
    this.fullFilledCallbacks = []; // 存储状态转变之前的注册的onFulfilled
    this.rejectedCallbacks = []; // 存储状态转变之前的注册的onRejected
    const resolve = (value) => {
      if(this.status === PENDING){
        this.status = FULLFILLED;
        this.value = value;
        this.fullFilledCallbacks.forEach(callback => callback(value));
      }
    }
    const reject = (reason) => {
      if(this.status === PENDING){
        this.status = REJECTED;
        this.reason = reason;
        this.rejectedCallbacks.forEach(callback => callback(reason));
      }
    }
    try{
      fn(resolve, reject);
    }catch(e){
      reject(e);
    }
  }

  then(onFulfilled, onRejected){
    if(typeof onFullfilled !== 'function'){
      onFullfilled = () => {};
    }
    if(typeof onRejected !== 'function'){
      onRejected = () => {};
    }

    if (this.status === 'fullfilled'){
      onFulfilled(this.value);
    } else if (this.status === 'rejected'){
      onRejected(this.reason);
    } else {
      this.fullFilledCallbacks.push(onFulfilled);
      this.rejectedCallbacks.push(onRejected);
    }
  }
}
```

以上代码很简单，我们借助`fullFilledCallbacks`和`rejectedCallbacks`两个数组存储在promise状态为发生转变之前通过`then`方法注册的`onFulfilled`和`onRejected`回调。

#### then的返回值

由PromiseA+规范可以，`then`方法必须返回一个promise：

```js
  promise2 = promise1.then(onFulfilled, onRejected);
```

且promise2的状态由`onFullfilled`或者`onRejected`的返回值决定，如何进行转换则由`promise resolution procedure`方法来实现。我们先不考虑`promise resolution procedure`，根据规范实现如下：

```js
  then(onFulfilled, onRejected){
    let _resolve;
    let _reject;
    const promise2 = new Promise((resolve, reject) => {
      _resolve = resolve;
      _reject = reject;
    });

    // 如果`onFulfilled`不是一个函数，且`promise1`为`fullfilled`状态，则`promise2`也转变为`fullfilled`,且结果值和`promise1`一样
    if(typeof onFullfilled !== 'function'){
      onFullfilled = (value) => value;
    }

    // 如果`onRejected`不是一个函数，且`promise1`为`rejected`状态，则`promise2`也转变为`rejected`,且失败原因和`promise1`一样
    if(typeof onRejected !== 'function'){
      onRejected = (reason) => throw new Error(reason);
    }

    // 如果`onFulfilled`或者`onRejected`返回`x`, 则运行Pomise Resolution Procedure`[[Resolve]](promise2, x)`
    const excuteFullfilled = () => {
      const x = onFulfilled(this.value);
      this.promiseResolution(promise2, x, _resolve, _reject);
    }
    const excuteRejected = () => {
      const x = onRejected(this.reason);
      this.promiseResolution(promise2, x, _resolve, _reject);
    }

    // 如果`onFulfilled`或者`onRejected`抛出一个异常`e`,`promise2`必须以`e`作为失败原因变成`rejected`状态
    try{
      if (this.status === 'fullfilled'){
        excuteFullfilled();
      } else if (this.status === 'rejected'){
        excuteRejected();
      } else {
        this.fullFilledCallbacks.push(() => {
          try{
           excuteFullfilled();
          }catch(e){
            _reject(e);
          }
        });
        this.rejectedCallbacks.push(() => {
          try{
            excuteRejected();
          }catch(e){
            _reject(e);
          }
        });
      }
    }catch(e){
      _reject(e);
    }

    return promise2;
  }

  promiseResolution(){
    ...
  }
```

接下来我们来实现`promiseResolution`函数，如果已经忘了可以去复习一下`promiseResolution`的规范。`promiseResolution`的主要任务是根据`onFulFilled`或者`onRejected`的返回值`x`来决定`promise2`的状态转变。

```js
promiseResolution(promise, x, resolve, reject){
  // 如果`resolvePromise`和`rejectPromise`都被调用了，或者以同样的参数被调用多次，则以第一次调用为准，忽略之后的调用
  let called = false;

  // 如果`resolvePromise`以参数`y`被调用的话，则执行`[[Resolve]](promise, y)`
  const resolvePromise = (y) => {
    if(!called){
      this.promiseResolution(promise, y, resolve, reject);
      called = true;
    }
  }

  // 如果`rejectPromise`以参数`r`被调用的话,则以`r`作为失败原因使``变成`rejected`状态
  const rejectPromise = (r) => {
    f(!called){
      reject(r);
      called = true;
    }
  }

  if(promise === x){
    throw new Error('TypeError');
  }

  // 如果`x`是一个promise，则`promise`采用`x`的状态以及相应的结果值或者原因
  if( x instanceof Promise){
    x.then(resovle, reject)
  }

  if( x && (typeof x === 'object' || typeof x === 'function')){
    let then = x.then;
    if(typeof then === 'function'){
      try{
        then.call(x, resolvePromise, rejectPromise);
      }catch(e){
        if(!called){
          reject(e);
        }
      }
    }else{
      // 如果`then`不是一个函数，则`promise`以`x`为结果值变为`fullfilled`状态
      resolve(x);
    }
  } else {
    // 如果`x`不是一个对象或者函数，则`promise`以`x`为结果值变为`fullfilled`状态
    resolve(x);
  }

}
```

#### 异步问题

规范中有一条时，`onFulfilled`或者`onRejected`直到执行环境堆栈只包含平台代码时才可以执行。规范给出的注释是：平台代码是指引擎、执行环境和Promise实现代码，就是说js主栈中不能有其他代码，这就是要求我们异步执行`onFulfilled`或者`onRejected`；我们可以用宏任务(setTimeout 或者 setImmediate)或者微任务（MutationObserver or process.nextTick）机制来实现。

我们知道，ES6的promise的then方法回调异步执行的机制是微任务。在浏览器环境我们可以使用[MutationObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver)来模拟微任务机制，如果浏览器不支持`MutationObserver`,我们回退到setTimeout使用宏任务实现。

```js
function isNative(fn){
  return fn.toString.includes('native code');
}

function asynTask(){
  if(typeof MutationObserver === 'function' && isNative(MutationObserver)){
    return (fn) => {
      var targetNode = document.createElement('div');
      var config = { attributes: true };
      // Create an observer instance linked to the callback function
      var observer = new MutationObserver(fn);
      // Start observing the target node for configured mutations
      observer.observe(targetNode, config);
      targetNode.id = 'anyway';
    }
  } else if(typeof setImmediate === 'function' && isNative(setImmediate)){
    return (fn) => {setImmediate(fn)};
  } else{
    return (fn) => {setTimeout(fn, 0)};
  }
}
class Promise{
  registerAsyn: asynExcute()
}
```

我们使用异步机制来重写`then`
```js
then(onFulfilled, onRejected){
    let _resolve;
    let _reject;
    const promise2 = new Promise((resolve, reject) => {
      _resolve = resolve;
      _reject = reject;
    });

    // 如果`onFulfilled`不是一个函数，且`promise1`为`fullfilled`状态，则`promise2`也转变为`fullfilled`,且结果值和`promise1`一样
    if(typeof onFullfilled !== 'function'){
      onFullfilled = (value) => value;
    }

    // 如果`onRejected`不是一个函数，且`promise1`为`rejected`状态，则`promise2`也转变为`rejected`,且失败原因和`promise1`一样
    if(typeof onRejected !== 'function'){
      onRejected = (reason) => throw new Error(reason);
    }

    // 如果`onFulfilled`或者`onRejected`返回`x`, 则运行Pomise Resolution Procedure`[[Resolve]](promise2, x)`
    const excuteFullfilled = () => {
      try{
        const x = onFulfilled(this.value);
        this.promiseResolution(promise2, x, _resolve, _reject);
      }catch(e){
        _reject(e);
      }
    }
    const excuteRejected = () => {
      try{
        const x = onRejected(this.reason);
        this.promiseResolution(promise2, x, _resolve, _reject);
      }catch(e){
        _reject(e);
      }
    }

    if (this.status === 'fullfilled'){
      registerAysn(excuteFullfilled);
    } else if (this.status === 'rejected'){
      registerAysn(excuteRejected);
    } else {
      this.fullFilledCallbacks.push(() => {
        registerAysn(excuteFullfilled);
      });
      this.rejectedCallbacks.push(() => {
        registerAysn(excuteRejected);
      });
    }

    return promise2;
  }

```
目前为止我们实现了Promise的基本功能。

#### Promise静态方法和其他实例方法

```js
class Promise{
  catch(fn) {
    return this.then((null, fn);
  }

  finally(fn) {
    const P = this.constructor;
    return this.then(
      value  => P.resolve(fn()).then(() => value),
      reason => P.resolve(fn()).then(() => { throw reason })
    );
  }
}

//resolve方法

Promise.resolve = function(value) {
  if (value instanceof Promise) {
    return value;
  }

  return new Promise(function(resolve) {
    resolve(value);
  });
};

Promise.reject = function(value) {
  return new Promise(function(resolve, reject) {
    reject(value);
  });
};

Promise.race = function(promises) {
  return new Promise(function(resolve, reject) {
    for (var i = 0, len = promises.length; i < len; i++) {
      promises[i].then(resolve, reject);
    }
  });
};

Promise.all = function(promises){
  if (!promises || typeof promises.length === 'undefined')
      throw new TypeError('Promise.all should accepts an array');
  let values = [];
  let i = 0;
  let length = promises.length;
  function processData(index,data){
    values[index] = data;
    i++;
    if(i == length){
      resolve(arr);
    };
  };
  return new Promise((resolve,reject)=>{
    for(let i=0;i<promises.length;i++){
      promises[i].then(data=>{
        processData(i,data);
      },reject);
    };
  });
}

```

### 参考来源

https://juejin.im/post/5b2f02cd5188252b937548ab#heading-8

https://github.com/taylorhakes/promise-polyfill