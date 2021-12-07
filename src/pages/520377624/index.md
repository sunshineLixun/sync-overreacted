---
title: '监听vuex中state的变化'
date: '2019-11-09'
spoiler: ''
---

  前端业务开发中会经常遇到这么一个场景：

某个组件UI改变了自身的属性：e.g:

![header](https://user-images.githubusercontent.com/15700015/68526732-e0467100-0319-11ea-8d0b-572c442ebd8a.png)


修改header组件中服务点的数值，而其他组件需要作出联动的数据变化，但是他们又不属于父-子（父-子-孙）关系。这时候就需要去监听`store`中`state`属性数值的修改。


![exContent](https://user-images.githubusercontent.com/15700015/68526841-0587af00-031b-11ea-97b5-c694cc6cba49.png)


给store中state一个服务点id变量

```jsx
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
    state: {
        serId: ''
    },
    mutations: {
        changeServeId (state, payload) {
            state.serId = payload
        }
    },
    actions: {
        //
    },
    modules: {

    }
})

```

当服务点修改时 `changeServeId` 改变store中state数值

在其他组件中监听state的变化

```jsx
    computed: {
        getSerId() {
            return this.$store.state.serId
        }
    }, 
    watch: {
        getSerId(val) {
            this.queryList()
        }
    },
```

`computed`返回`serId`变化后的值，`watch`监听`getSerId`的变化，这样就能拿到最新的`serId`去做业务请求了。






  