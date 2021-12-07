---
title: 'Vue-Router源码分析（一）： install过程'
date: '2021-12-05'
spoiler: ''
---

  ### Vue-router插件install过程。

通常使用vue-router管理SPA 路由时，一定要把router实例挂载到根组件上：

![carbon (4)](https://user-images.githubusercontent.com/15700015/144749742-035d3b6f-7b8c-4270-8591-a21b5bc2066e.png)

在各个组件中可以直接使用 `this.$router`  `this.$route` 来访问当前路由信息以及router实例。Vue-Router是怎么做到这一点的？

Vue规定：如果使用Vue插件，则一定要提供install方法，Vue.use方法会把Vue本身传递到插件中，插件作者只需要提供install方法即可获取到Vue类本身。插件可以使用Vue中定义的各种方法，包括prototype中定义的方法。

看下VueRouter install方法做了什么。

```jsx
export let _Vue

export function install (Vue) {
  // 防止多次注册
  if (install.installed && _Vue === Vue) return

  // 插件注册标识符
  install.installed = true

  // 保存Vue 类的引用。
  _Vue = Vue

  const isDef = v => v !== undefined

  const registerInstance = (vm, callVal) => {
    let i = vm.$options._parentVnode
    if (isDef(i) && isDef(i = i.data) && isDef(i = i.registerRouteInstance)) {
      i(vm, callVal)
    }
  }

  Vue.mixin({
    beforeCreate () {
      if (isDef(this.$options.router)) { // 根组件
       // _routerRoot 属性指向本身
        this._routerRoot = this
       //  保存用户传入进来的router对象
        this._router = this.$options.router
       // router初始化
        this._router.init(this)
        Vue.util.defineReactive(this, '_route', this._router.history.current)
      } else {
        // 子组件 获取父组件_routerRoot，为了拿到父组件中_router对象
        this._routerRoot = (this.$parent && this.$parent._routerRoot) || this
      }
      registerInstance(this, this)
    },
    destroyed () {
      registerInstance(this)
    }
  })

  Object.defineProperty(Vue.prototype, '$router', {
    get () { return this._routerRoot._router }
  })

  Object.defineProperty(Vue.prototype, '$route', {
    get () { return this._routerRoot._route }
  })

  Vue.component('RouterView', View)
  Vue.component('RouterLink', Link)

  const strats = Vue.config.optionMergeStrategies
  // use the same hook merging strategy for route hooks
  strats.beforeRouteEnter = strats.beforeRouteLeave = strats.beforeRouteUpdate = strats.created
}

```

第一步：
为了防止多次注册插件，做了一个判断条件：`install.installed && _Vue === Vue` 当已经注册过该组件并且内部_Vue 已经赋值过，这不进行后续操作。

第二步：
标记组件已经被注册，_Vue保存对传经来的Vue类本身的引用，后续方便其他文件使用Vue中的方法。

第三步：
通过混入的方式，在beforeCreate生命周期函数中，对$options属性中传入的router做处理。
当程序第一次进入时，一定会走`this.$options.router` 这个判断中，因为根实例中一定会有用户传入的router对象。紧接着在根实例中保存一个变量_routerRoot 对自身的引用，_router 属性则是用户传入进来的router路由信息对象。

这里有个知识点，因为组件是一层层渲染的，beforeCreate钩子函数执行的顺序是：根组件 - > 父组件 - > 子组件 -> 孙组件 ....。 以此下去。所当子孙组件渲染是，执行到beforeCreate 时就会进入else 判断。 else判断中主要做了1件事：

1. 申明_routerRoot 属性，this.$parent && this.$parent._routerRoot  这个意思就是如果有父组件并且父组件中有_routerRoot属性，当前子组件中的_routerRoot 就会保存this.$parent._routerRoot，而父组件中的_routerRoot 又是他本身，所有在子组件中_routerRoot 就是父组件自身，如果没有父组件，则_routerRoot属性 就指向子组件this自身。

如果有孙组件，道理一样，那么孙组件一定会走到 else 循环，那么孙组件的_routerRoot 就是指向它父组件的_routerRoot。

所以整个_routerRoot  指向过程是这样：

孙组件 _routerRoot  ->  子组件 _routerRoot  -> 父组件_routerRoot -> 根组件_routerRoot

第四步：

```jsx
  Object.defineProperty(Vue.prototype, '$router', {
    get () { return this._routerRoot._router }
  })

  Object.defineProperty(Vue.prototype, '$route', {
    get () { return this._routerRoot._route }
  })
```

像Vue原型上注册 $router  和  $route 属性，并只提供一个getter方法。其中返回的是当前组件中父组件的_router 和 _route 属性，这里也可以这么理解，_routerRoot 最终指向的是根组件的_routerRoot，_router 和 _route 也就是 根组件定义的。

到了第四步这里 基本上已经弄清楚 项目中每个Vue组件是怎么获取到 $router  和$route 了。

第5步：
```jsx
  Vue.component('RouterView', View)
  Vue.component('RouterLink', Link)
```
注册RouterView和RouterLink组件。


其实类推到Vuex中 也是这么一个处理过程，每个组件都能获取到$store对象，实现思路是一样的。





  