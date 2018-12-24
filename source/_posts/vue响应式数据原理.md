---
title: vue响应式数据原理
date: 2018-12-18 22:17:15
description: vue和react是现在前端框架的双子星。vue以其简单好用而闻名。vue以数据驱动视图，数据响应系统是vue的核心。这篇文章主要是结合源码分析vue响应式系统的原理和实现。
categories: 前端
tags: vue 数据响应系统
---
### 代理

下面这段代码是vue使用的典型方式：

```html
<div id="app-5">
  <p>{{ message }}</p>
  <button v-on:click="reverseMessage">逆转消息</button>
</div>
```
```js
var app5 = new Vue({
  el: '#app-5',
  data: {
    message: 'Hello Vue.js!'
  },
  methods: {
    reverseMessage: function () {
      this.message = this.message.split('').reverse().join('')
    }
  }
})
```
![base](/images/vue_data/base.gif)

我们可以看到当我们给`this.message`赋值时，视图会自动更新。这就是vue数据响应系统做的事情，当然vue做的事情远比这个复杂地多，但这是核心理念。原理也很简单，vue帮我们代理了数据的赋值操作，在数据赋值时进行了DOM的更新，这些对于vue使用者是不可见的，也是无需考虑的。

实现数据代理在js中有两种方式，一个是ES5的`getter`和`setter`方法；一个是ES6的`proxy`api。vue2.5及以下采用的是`getter`和`setter`方法；即将出来的vue3.0全部改为`proxy`方式来实现。

#### getter和setter

Object提供一个方法[definePropery](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)可以让我们给一个对象的属性定义`getter`和`setter`，从而代理对象属性的取值和赋值操作，用法如下：

```js
var o = {};
Object.defineProperty(o, "b", {
  get: function(){
    alert('我在取值');
  },
  set: function(newValue){
    alert('我在赋值');
  },
  enumerable : true,
  configurable : true
});
var a = o.b; // 弹出“我在取值”
o.b=5; // 弹出“我在赋值”
```
有了这个特性，我们就可以实现最简单的双向数据绑定。比如vue中的v-model效果。

```html
<input type="text" id="name"></input>
```
```js
var data = {};
var nameDom = document.querySelector('#name');
Object.defineProperty(data, 'value', {
  get: function(){
    return nameDom.value;
  },
  set: function(value){
    nameDom.value = value;
  }
})
```
![model](/images/vue_data/model.gif)
由此我们便实现了最简单的双向数据绑定。当然，vue实现响应式数据的思路和上面不一样，要复杂的多，但是基本理念就是通过劫持数据的赋值和取值操作来完成的。

#### 观察者模式

我们要想实现vue的响应式数据，就要给vue初始化对象的data和props的所有属性设置setter和getter函数。vue源码`src/core/instance/state.js`的`initData`函数有如下代码，其中`observe`都是循环遍历data对象，给每个属性都设置setter和getter。
```js
  // src/core/instance/state.js
  // observe data
  observe(data, true /* asRootData */)
```
我们可以知道的是setter函数中的操作肯定是要更新DOM的，那每个数据绑定的DOM不同，绑定的属性也不同，如果像我们上面那样把每个响应式数据和具体DOM的属性绑定起来，就太麻烦和复杂了。Vue采取的策略是`虚拟DOM`，每次数据setter操作，都会根据你写的`template`或者`render`生成`虚拟DOM`，然后和之前的`虚拟DOM`进行比较，如果有不同，则进行DOM更新，而且只更新有变化的部分；如果相同就不做操作。这种方式可以解决我们的问题，性能也没有问题。在vue实例mount的过程中会执行以下代码：
```js
updateComponent = () => {
  vm._update(vm._render(), hydrating)
}
```

这句就是用来执行DOM的更新渲染的，我们在响应式数据的setter中应该执行`updateComponent`就能达到我们的目的了。看起来很简单是吧，但是现在有两个问题：

1. 我们在data和props中声明的属性不一定都绑定到Dom上了，如果是没有绑定到Dom上的数据，在进行setter的时候，也要DOM更新操作，虽然不会引起真正的DOM更新，但也是很浪费性能。
2. 我们数据setter可能会不止有Dom更新的任务，比如watch了一个属性，那么这个属性就有Dom更新和watch绑定的回调两个任务。

数据setter绑定不同的任务，在数据改变时，执行所有绑定的任务，这个不就是观察者模式嘛。。。

观察者模式一个典型的例子就是DOM元素的事件绑定

```js
var btnDom = document.getElementById('btn');
btnDom.addEventListener('click', function(){
  console.log('click事件发生了，做点啥...');
});
```
我们给btnDom绑定`click`事件，就相当于在`观察`btnDom，当btnDom被点击，就会调用我们绑定的事件。现在我们的响应式数据（data和props）就是我们观察的对象，我们把需要执行的任务放入响应式数据的setter里，在响应式数据被赋值的时候，执行这些任务。观察者模式这个名字是和现实的一个类比，有的有`观察者`实例，有的直接注册`回调函数`，其实本质是一样的，这些`观察者实例`或者`回调函数`都在观察的动作中被放入`被观察者`的实例中的，在`被观察者`发生改变时，执行注册在自己身上的`回调函数`或者通知`观察者`。下面是一段典型的观察者模式实现：

 ```js

//被观察者
class Subject{
  constructor(){
    this.observerList = [];
  }

  addObserver(observer){
    this.observerList.push(observer);
  }

  removeObserver(observer){
    const index = this.observerList.findIndex(item => item === observer);
    if(index !== -1){
      this.observerList.splice(index, 1);
    }
  }

  notify(context){
    const length = this.observerList.length;
    for(let i = 0; i<length; i++){
      const observer = this.observerList[i];
      observer.notify(context);
    }
  }
}

//观察者
class Observer{
  constructor(){
    this.notify = function(){
      // ...
    };
  }
}
 ```
 Subject实例通过`addObserver`和`removeObserver`来添加和删除观察者，然后在合适的时机通过`notify`方法通知所有的观察者。vue也是类似的实现机制，不过Vue的设计比较巧妙，实现形式有所不同。vue有3个类是用来处理响应式数据的观察者模式：`Observer`、`Watcher`、`Dep`。

 - Observer， 该类主要作用是用来定义属性的getter和setter方法
 - Watcher， 观察者类，且同时用于$watch实例方法和watch指令
 - Dep, 观察者容器，每一个响应式数据的属性都拥有一个自己独立的Dep实例，盛放自己的观察者

我们上面写的观察者模式或者是事件绑定，需要我们主动去添加`观察者`，那么在响应式数据这个模式当中我们应该在何时去收集属性自己的观察者那？答案是在响应式数据的`getter`中收集。因为被观察的数据在求值的时候肯定会触发`getter`函数，这是一个很好的时机，而且也能避免没有参与DOM更新的属性被绑定DOM更新的`观察者`。所以，vue采用的方式是在`getter`中收集`观察者`，在`setter`中通知`观察者`。

`Observer`类型中有一个`defineReactive`函数，这个函数主要是用来定义属性的getter和setter方法,下面是一个简化版的`defineReactive`函数，去掉了一个边界情况的数据，只考虑对象这种响应式数据。
 ```js
 /**
 * Define a reactive property on an Object.
 */
 export function defineReactive (
  obj: Object,
  key: string,
  shallow?: boolean
) {
  const dep = new Dep()
  val = obj[key];
  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      if (Dep.target) {
        dep.depend()
      }
      return val
    },

    set: function reactiveSetter (newVal) {
      /* eslint-disable no-self-compare */
      if (newVal === val || (newVal !== newVal && val !== val)) {
        return
      }
      val = newVal
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}
 ```
上面代码还是很清楚简单的，在`getter`中的`dep.depend`就是收集`观察者`；在`setter`中`dep.notify`就是通知`观察者`。每一个响应式属性都拥有自己的闭包`Dep`实例，这个`Dep`实例中装载这所有该属性的`观察者`。那么`Dep.target`是什么东西那？它就是我们所要收集的观察者。这里`Dep.target`可能会有点迷糊，我们先来考虑一下vue的$watch实例方法或者watch指令是怎么用的：

```js
vm.$watch(expOrFn, function(){...});
```
当`expOrFn`的值发送变化时，执行回调函数。而`$watch`方法就是新建了一个`Watcher`实例:

```js
Vue.prototype.$watch = function (
  expOrFn: string | Function,
  cb: any,
  options?: Object
): Function {
  const vm: Component = this
  if (isPlainObject(cb)) { // 用于处理watch指令cb是一个对象，带有immediate或者deep参数的情况，createWatcher中是整理参数，创建Watcher实例
    return createWatcher(vm, expOrFn, cb, options)
  }
  options = options || {}
  const watcher = new Watcher(vm, expOrFn, cb, options)
  if (options.immediate) {
    cb.call(vm, watcher.value)
  }
  return function unwatchFn () {
    watcher.teardown()
  }
}
```
现在我们需要明确的一点就是，其实通过改变数据来更新DOM这个操作，其实就是创建了一个渲染函数的`Watcher`实例，跟我们使用`$watch`方法去观察一个数据是一样的，只不过这个操作是Vue主动做的，只不过它观察的是所有`<template>`或者`render`中的数据。下面这段代码就是创建渲染函数的`Watcher`实例：

```js
new Watcher(vm, updateComponent, noop, {
  before () {
    if (vm._isMounted) {
      callHook(vm, 'beforeUpdate')
    }
  }
}, true /* isRenderWatcher */)
```
`updateComponent`函数上面已经说过，是用来更新DOM操作的。我们先来看一下Watcher类构造函数的参数，`Watcher(vm, expOrFn, cb, options)`一共有4个：

- `vm`, Vue实例
- `expOrFn`, 被观察的表达式或者函数，用来求值同时触发被观察数据的`getter`函数，用于收集该观察者，所以我们上面说的`Dep.target`就是在`expOrFn`求值之前被赋值为该观察者实例的
- `cd`,回调函数，被观察数据改变时，执行的回调函数
- `options`, 一些参数设置
- `isRenderWatcher`,是否为渲染函数的观察者

那我们看这个`渲染函数`的观察者实例的构造参数就有些奇怪，因为它的回调是`noop`,`noop`是定义了一个空函数。这不是很奇怪吗，在数据变化的时候什么都不做，怎么更新DOM？实际上在数据发生变化的时候，Watcher实例都要对`expOrFn`重新求一遍值，这样才能知道`expOrFn`的值有没有变化，进而决定是否要执行`cb`;而`updateComponent`这个更新DOM的函数同时满足触发`getter`和更新DOM的需求，所以在这里就不需要设置`cb`了，同样如果我们在编码时，有这样同时满足收集依赖和满足回调的函数，也可以这样用。

### 避免重复收集观察者

那每次数据发生变化的时候，`观察者`都对`expOrFn`求值，岂不是每次都会触发`getter`函数，造成依赖重复收集？的确会，而且即便在一次求值过程中，也可能触发同一个数据多次（比如同一个属性出现在模板多个地方），不过Vue已经实现了避免收集重复依赖的处理: 

```js
class Watcher{
  addDep (dep: Dep) {
    const id = dep.id
    if (!this.newDepIds.has(id)) {
      this.newDepIds.add(id)
      this.newDeps.push(dep)
      if (!this.depIds.has(id)) {
        dep.addSub(this)
      }
    }
  }
}

class Dep{
  addSub (sub: Watcher) {
    this.subs.push(sub)
  }

  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }
}
```
上述代码就是Vue用来避免收集重复依赖的，我们知道，在响应式数据的`getter`中，我们会调用该属性所拥有的`Dep`实例的`depend`方法来收集`观察者`。我们可以看到在`depend`方法中调用了观察者实例方法`addDep`,而在`addDep`方法中我们可以看到又调用了`Dep`的实例方法`addSub`给`Dep`这个容器加入`观察者`。有点绕，两个类来回调用，目的只有一个，就是避免收集重复的`观察者`。

我们可以看到`addDep`方法中有3个值，决定了是否添加`观察者`，我们先来说明一下这3个值的作用：

- newDepIds，本次求值所收集的`Dep`实例Id列表，用来避免本次求值的重复收集，在每次求值完成之后都会被赋值给depIds,然后被清空
- depIds，上次求值所收集的`Dep`实例Id列表，用来避免两次求值之间的重复收集以及去除废弃的`观察者`
- newDeps, 本次求值所收集的`Dep`实例列表, 在每次求值完成之后都会被赋值给deps,然后被清空

这样看代码的逻辑就很清楚了，先判断在本次求值中是否已经收集了该`Dep`实例，如果没有，则将该`Dep`的id添加到`newDepIds`,然后再判断，该`Dep`实例是否存在于上次求值的`Dep`实例Id列表，如果没有，则将该`观察者`放入`Dep`实例中，作为该`Dep`实例所属的响应式数据所拥有的`观察者`。

在每次求值之后，都会执行`cleanupDeps`,用于给`newDepIds`赋值给`depIds`，清空`newDepIds`, 并且去除废弃的`观察者`，下面这段代码就是去除废弃的`观察者`：

```js
cleanupDeps () {
  let i = this.deps.length
  while (i--) {
    const dep = this.deps[i]
    if (!this.newDepIds.has(dep.id)) {
      dep.removeSub(this)
    }
  }
  // 省略...
}
```

凡是在上次求值过程中存在的`Dep`实例，在本次求值中不存在了，说明该`Dep`实例已经被废弃了，更直白的说法就是该`Dep`实例所属的响应式数据已经不在本次求值过程中了，需要把该`观察者`在`Dep`实例中去除。

### 异步更新队列

在同一个js`任务队列`中，我们可能改变多个模板中的响应式数据，这样会造成多次触发渲染函数的`观察者`，造成多次重复地渲染DOM，造成性能浪费。解决这个问题的办法在于我们要有一个合适的时机统一处理一个js`任务队列`中所有被触发的`观察者`，对于重复的`观察者`只执行一次。这个合适的时机就是`微任务`,在js`任务队列`执行完之后，会立即执行在本次`任务队列`中产生的所有微任务。且在两次js`任务队列`之间会穿插着DOM更新，所以在`微任务`中把所有相关的数据更新，是最优的。下面`queueWatcher`是异步模式下`观察者`被通知时执行的操作，`queue`存放在本次`任务队列`中所有被通知的`观察者`，`nextTick`是Vue实现的`微任务`机制（在不支持微任务的情况下，回退到`宏任务`）,`flushSchedulerQueue`则是用来执行`queue`中的`观察者`,并清空`queue`：
```js
export function queueWatcher (watcher: Watcher) {
  const id = watcher.id
  if (has[id] == null) {
    has[id] = true
    queue.push(watcher)
    // queue the flush
    if (!waiting) {
      waiting = true

      if (process.env.NODE_ENV !== 'production' && !config.async) {
        flushSchedulerQueue()
        return
      }
      nextTick(flushSchedulerQueue)
    }
  }
}
```

