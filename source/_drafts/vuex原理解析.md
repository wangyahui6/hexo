---
title: vuex原理解析
date: 2018-12-29 09:32:04
description: Vuex是借鉴了Flux,Redux，量身为Vue打造的状态管理系统。实现了自动注入$store，响应式数据，模块化，状态变更的单一来源和记录，即插即用。
categories: 前端
tags: vue
---

### 状态管理

在现在前端`组件化`开发的方式中，组件之间的状态管理无疑已经成了一个影响开发方式的重要问题。为了保持状态清晰，vue和react都采用了自上而下的单项状态传递。然后真正出现状态管理问题的是以下两种情况：

- 嵌套很深的组件之间的状态传递。重重重孙辈的组件需要祖先的状态，要一辈一辈地传递过来。写的时候很麻烦，而且修改起来更是噩梦。如果你要改变这个状态，那就要通过事件一层一层的传递上去，所以。。。
- 兄弟组件之间的状态传递。咦？这怎么传？我的状态凭什么给你。。。然后兄弟俩打起来了。React提出一种解决方案，叫变量提升，把这个兄弟两个都需要的状态放到父组件中，由爸爸共享给两个儿子。这种情况还好一些。

vue官网给出最简单的解决方案就是所有组件的data都使用同一个对象实例，利用对象引用相同来实现，如下：
```js
const sourceOfTruth = {}

const vmA = new Vue({
  data: {
    sharedState: sourceOfTruth,
    ...
  }
})

const vmB = new Vue({
  data: {
    sharedState: sourceOfTruth,
    ...
  }
})
```
这样就可以轻松解决这个共享问题了，其实本质上的东西很简单。但是这种多层嵌套组件共享状态的问题最令人头痛的一个问题，是不可知性，在组件树中，每一个组件都可能会更改sharedState的状态，一旦sharedState出问题了，我们解决起来都无从入手。所以我们需要知道修改sharedState的修改记录。改进版的解决方案如下：

```js
var store = {
  debug: true,
  state: {
    message: 'Hello!'
  },
  setMessageAction (newValue) {
    if (this.debug) console.log('setMessageAction triggered with', newValue)
    this.state.message = newValue
  },
}

var vmA = new Vue({
  data: {
    privateState: {},
    sharedState: store.state
  },
  methods: {
    changeMessage() {
      store.setMessageAction('我要改变你');
    }
  }
})

var vmB = new Vue({
  data: {
    privateState: {},
    sharedState: store.state
  }
})
```

我们采用单一`store`的模式来管理`state`,对于store中`state`的改变只能通过`setMessageAction`,不允许直接修改`state`,确保改变的单一来源，同时记录日志。这样出问题的时候我们就很清楚地知道，什么类型的数据被改变了以及如何被触发的。

![state](/images/vuex/state.png)

其实Vuex的根本思想跟上面所说的一样，只不过Vuex加了很多功能，方便使用和调试。下面我们就来看看Vuex都实现了哪些功能。

### Vuex解析

我们先来了解一下Vuex的设计和用法：

![vuex](/images/vuex/vuex.png)

上面这个状态图清晰地表示了vuex的工作方式，vuex的`state`用来渲染组件，组件通过用户交互或者自发逻辑发起`dispatch`,引发`action`的执行，`action`可以试同步操作，也可以是异步请求，通常`action`都会提交`commit`,触发`mutation`的执行。注意，`mutation`是唯一改变`state`的合法手段。这里有一点是图里没有画出来的，就是组件中也是可以直接提交`commit`,触发`mutation`执行的。

vuex的使用方法很简单，第一步，新建store实例：

```js
import Vuex from 'vuex';

export default new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})
```
第二步，在Vue根组件中注入store：

```js
import Vue from 'vue';
import Vuex from 'vuex';
import store from 'store.js';

Vue.use(Vuex);
new Vue({
  data:{},
  template:`<div>{{count}}</div>`,
  computed: {
    count() {
      return this.$store.state.count;
    }
  },
  methods: {
    add() {
      this.$store.dispatch('increment');
    }
  },
  store
})
```

现在我们所有的组件内都可以通过`this.$store`来使用`vuex`了。

#### $store的注入

首先我们来解决一个简单的问题，为什么我们使用了`Vue.use(Vuex)`之后，我们所有的组件都可以使用`this.$store`了?

`Vue.use`是vue提供的插件机制。所谓插件，就是注册一些全局可用的命令，通常有以下几种情况：

- 添加全局方法或者属性，比如`Vue.customElement`
- 添加全局资源：指令/过滤器/过渡等
- 通过全局 mixin 方法添加一些组件选项
- 添加 Vue 实例方法，通过把它们添加到 Vue.prototype 上实现

使用方法也很简单, 需要注意的是插件应该在组件实例化之前就安装，否则会出现一些命令不生效，比如mixins的created：

```js
MyPlugin.install = function (Vue, options) {
  // 1. 添加全局方法或属性
  Vue.myGlobalMethod = function () {
    // 逻辑...
  }

  // 2. 添加全局资源
  Vue.directive('my-directive', {
    bind (el, binding, vnode, oldVnode) {
      // 逻辑...
    }
    ...
  })

  // 3. 注入组件
  Vue.mixin({
    created: function () {
      // 逻辑...
    }
    ...
  })

  // 4. 添加实例方法
  Vue.prototype.$myMethod = function (methodOptions) {
    // 逻辑...
  }
};

// Vue.use会默认执行MyPlugin的install方法
Vue.use(MyPlugin);

```
其实这些全局命令我们可以在任何地方和方式注册，不一定通过`Vue.use`。`Vue.use`是提供了一种在Vue中使用插件的规范，这样有利于团队合作，而且Vue.use也做了不会重复安装的措施。

`Vuex`本质上就是一个插件，这个插件在所有组件上都注入了`$store`。我们来看一下`Vuex`的install方法：

```js
let Vue;
export function install (_Vue) {
  if (Vue && _Vue === Vue) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      )
    }
    return
  }
  Vue = _Vue
  applyMixin(Vue)
}
```
首先判断了Vuex是否已经被安装过了，如果已经安装过给出错误提示。实际上`Vue.use`也做了防止重复安装的操作，这里应该是想提示用户错误。接下来看看`applyMixin`:

```js
export default function (Vue) {
  // Vue版本大于2的情况
  Vue.mixin({ beforeCreate: vuexInit })
  /**
   * Vuex init hook, injected into each instances init hooks list.
   */

  function vuexInit () {
    const options = this.$options
    // store injection
    if (options.store) {
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store
    }
  }
}
```
这里为了逻辑更加清楚，只保留了Vue版本大于2的情况，源码中兼容了2以下没有mixin的情况。我们可以看到实际上注册了全局的mixin，给每个组件都添加了一个`created`钩子函数。这个钩子函数判断了vue实例中是否有`store`属性，如果有的话，将`store`赋值给`this.$store`;如果没有`store`属性，则查看父组件是否有`$store`属性，如果有，则赋值给`this.$store`。从而实现了`$store`对所有组件的注入。其实这里说`所有组件`是不准确的，应该说添加了`store`属性的组件及其所有子孙组件。

#### actions、dispatch

首先在`Store`构造函数当中，会根据actions参数遍历注册action，注册函数如下所示：

```js
function registerAction (store, type, handler, local) {
  const entry = store._actions[type] || (store._actions[type] = [])
  entry.push(function wrappedActionHandler (payload, cb) {
    let res = handler.call(store, {
      dispatch: local.dispatch,
      commit: local.commit,
      getters: local.getters,
      state: local.state,
      rootGetters: store.getters,
      rootState: store.state
    }, payload, cb)
    if (!isPromise(res)) {
      res = Promise.resolve(res)
    }
    if (store._devtoolHook) {
      return res.catch(err => {
        store._devtoolHook.emit('vuex:error', err)
        throw err
      })
    } else {
      return res
    }
  })
}
```
我们可以看到，所有的action都存储在`store._actions`中，而且同一种类型的action可以注册多次，回调都被推入到数组中，在调用的时候统一调用。而且需要注意的一点是，actions的hanlder返回的是一个Promise。这主要是为了让action处理异步操作。

下面我们来看一下`dispatch`是如何触发`action`的执行的：

```js
dispatch (_type, _payload) {
    // check object-style dispatch
    const {
      type,
      payload
    } = unifyObjectStyle(_type, _payload)

    const action = { type, payload }
    const entry = this._actions[type]
    if (!entry) {
      if (process.env.NODE_ENV !== 'production') {
        console.error(`[vuex] unknown action type: ${type}`)
      }
      return
    }

    this._actionSubscribers.forEach(sub => sub(action, this.state))

    return entry.length > 1
      ? Promise.all(entry.map(handler => handler(payload)))
      : entry[0](payload)
  }
```
首先处理一下参数格式，然后取出相应类型的`actions`去执行, 这里我们可以看到，如果这种类型的`handler数组`长度大于1时，会被`Promise.all`包装并返回。下面看一个例子，说明这种情况：

```js
{
  actions: {
    actionA() {
      return getData1(); // asyn api
    },
    actionA() {
      return getData2(); // asyn api
    }
  }
}

store.dispatch('actionA').then(() = > {
  alert('操作成功'); // getData1和getData2都成功的时候才会执行
});
```

#### commit、mutations
我们先来看一下注册`mutation`的方法：

```js
function registerMutation (store, type, handler, local) {
  const entry = store._mutations[type] || (store._mutations[type] = [])
  entry.push(function wrappedMutationHandler (payload) {
    handler.call(store, local.state, payload)
  })
}
```

我们可以看到`mutioans`的回调函数也是一个数组，我们也可以注册多个同名的mutation。

```js

commit (_type, _payload, _options) {
    // check object-style commit
    const {
      type,
      payload,
      options
    } = unifyObjectStyle(_type, _payload, _options)

    const mutation = { type, payload }
    const entry = this._mutations[type]
    if (!entry) {
      if (process.env.NODE_ENV !== 'production') {
        console.error(`[vuex] unknown mutation type: ${type}`)
      }
      return
    }
    this._withCommit(() => {
      entry.forEach(function commitIterator (handler) {
        handler(payload)
      })
    })
    this._subscribers.forEach(sub => sub(mutation, this.state))

    if (
      process.env.NODE_ENV !== 'production' &&
      options && options.silent
    ) {
      console.warn(
        `[vuex] mutation type: ${type}. Silent option has been removed. ` +
        'Use the filter functionality in the vue-devtools'
      )
    }
  }
```

核心代码也很简单，取出该类型的`entry`,然后循环执行，这其中循环执行的代码被传入了`this._withCommit`,我们来看一下`this._withCommit`是干嘛用的：

```js
  _withCommit (fn) {
    const committing = this._committing
    this._committing = true
    fn()
    this._committing = committing
  }
```

这段代码的作用就是保证在`commit`的回调执行期间，`this._committing`为`true`。`this._committing`其实就是保证只有`mutation`能改变`state`的标志位。

```js
  // enable strict mode for new vm
  store._vm = new Vue({
    data: {
      $$state: state
    },
    computed
  })
  if (store.strict) {
    enableStrictMode(store)
  }

  function enableStrictMode (store) {
    store._vm.$watch(function () { return this._data.$$state }, () => {
      if (process.env.NODE_ENV !== 'production') {
        assert(store._committing, `do not mutate vuex store state outside mutation handlers.`)
      }
    }, { deep: true, sync: true })
  }
```

我们可以看到，在`vuex`中专门建了一个Vue实例`_vm`，该vue实例有两个作用，一个是利用计算属性`computed`来实现`getters`,二是在开发环境用法监听`state`的变化，在`store._committing`为false期间，提示修改非法。

#### getters
```js
function registerGetter (store, type, rawGetter, local) {
  if (store._wrappedGetters[type]) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(`[vuex] duplicate getter key: ${type}`)
    }
    return
  }
  store._wrappedGetters[type] = function wrappedGetter (store) {
    return rawGetter(
      local.state, // local state
      local.getters, // local getters
      store.state, // root state
      store.getters // root getters
    )
  }
}
```
```js
function resetStoreVM (store, state, hot) {
  const oldVm = store._vm

  // bind store public getters
  store.getters = {}
  const wrappedGetters = store._wrappedGetters
  const computed = {}
  forEachValue(wrappedGetters, (fn, key) => {
    // use computed to leverage its lazy-caching mechanism
    computed[key] = () => fn(store)
    Object.defineProperty(store.getters, key, {
      get: () => store._vm[key],
      enumerable: true // for local getters
    })
  })

  // use a Vue instance to store the state tree
  // suppress warnings just in case the user has added
  // some funky global mixins
  const silent = Vue.config.silent
  Vue.config.silent = true
  store._vm = new Vue({
    data: {
      $$state: state
    },
    computed
  })
  Vue.config.silent = silent

  // enable strict mode for new vm
  if (store.strict) {
    enableStrictMode(store)
  }

  if (oldVm) {
    if (hot) {
      // dispatch changes in all subscribed watchers
      // to force getter re-evaluation for hot reloading.
      store._withCommit(() => {
        oldVm._data.$$state = null
      })
    }
    Vue.nextTick(() => oldVm.$destroy())
  }
}
```