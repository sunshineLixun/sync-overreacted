---
title: 'Vue-Router源码分析(二): 路由映射表创建'
date: '2021-12-23'
spoiler: ''
---

  ```jsx
const router = new VueRouter({
  routes: [
    {
      path: '/',
      component: Home
    },
    {
      path: '/user',
      component: User,
      children: [
        {
          path: 'profile',
          component: UserProfile
        },
        {
          path: 'posts',
          component: UserPosts
        },
        {
          path: 'about',
          component: About
        }
      ]
    }
  ]
})
```
在我们使用Vue-Router配置路由信息时，这段代码是最常见的，routes对象接受一个对象数组，其中每个对象代表着一个路由信息，最核心就是path、component属性： `path`字段表示location的路由信息，`component`表示改路由对应的组件，当然还有一些其他属性，name、meta等等。

根据以上配置，Vue-Router在初始化时做了以下操作，保证path 一定能对应指定的component组件。

### 树的扁平化

在源码中：src/create-route-map.js文件中，将用户传递进来的routes属性遍历执行`addRouteRecord`方法

```jsx
routes.forEach(route => {
   addRouteRecord(pathList, pathMap, nameMap, route, parentRoute)
})
```

该方法中声明了一个`RouteRecord`对象，用来存储遍历的路由信息：

```jsx
  const record: RouteRecord = {
    path: normalizedPath, // 路由信息
    regex: compileRouteRegex(normalizedPath, pathToRegexpOptions), // 路由正则
    components: route.components || { default: route.component }, // 路由对应的组件
    alias: route.alias
      ? typeof route.alias === 'string'
        ? [route.alias]
        : route.alias
      : [], // 路由别名
    instances: {}, 
    enteredCbs: {}, // 进入组件时的回调
    name, //路由name
    parent, // 路由对应的父组件，这个属性很重要, 在递归是找到父组件的依据
    matchAs,
    redirect: route.redirect, // 重定向
    beforeEnter: route.beforeEnter, //钩子函数
    meta: route.meta || {}, //元信息
    props:
      route.props == null
        ? {}
        : route.components
          ? route.props
          : { default: route.props } //组件props
  }
```

接着判断route是否有children配置属性，执行递归操作。

```jsx
  if (route.children) {

   ......

    route.children.forEach(child => {

      ......

      // 递归执行，
      addRouteRecord(pathList, pathMap, nameMap, child, record, childMatchAs)
    })
  }
```

这里有个点需要注意，

- 在每次递归是都会去判断当前路由信息是否有`parent`，如果存在就要去拼接`parent`路由`path`和当前子路由`path`路劲，具体在`normalizePath`方法中:

```jsx
function normalizePath (
  path: string,
  parent?: RouteRecord,
  strict?: boolean
): string {
  // 正则相关
  if (!strict) path = path.replace(/\/$/, '')  
  if (path[0] === '/') return path //判断是否是根路由信息， "/"
  if (parent == null) return path // 不存在parent，说明当前 path就是为第一级路由信息，返回当前path路径即可
  return cleanPath(`${parent.path}/${path}`) // 存在，说明当前路由为childeren配置项的路由信息，拼接父路由和子路由path
}
```
那parent录音信息是在哪里传入的？
是在遍历第一次的路由信息时传入的。可以看递归时传入的参数：`addRouteRecord(pathList, pathMap, nameMap, child, record, childMatchAs)` 。child参数即为当前子路由是所归属的父路由信息。也就是上面声明的`RouteRecord `对象。

经过递归之后，`record`对象就会是这个样子

```jsx
 record = {
   path: '/',
   component: <Home>
 }
 record = {
   path: '/user/profile',
   component: <UserProfile>
 }
 record = {
   path: '/user/posts',
   component: <UserPosts>
 }
 record = {
   path: '/user/about',
   component: <About>
 }
```

在`createRouteMap`方法中会有一个`pathMap`对象用来保存扁平化之后的路由对象信息。

```jsx
  // 先读取oldPathMap 的值，oldPathMap是动态路由添加方法`addRoutes`提供的参数，oldPathMap就是在项目初始化声明的静态路由配置信息。项目初始化时肯定没有oldPathMap，所以创建了一个空对象。
  const pathMap: Dictionary<RouteRecord> = oldPathMap || Object.create(null)
```

回到`addRouteRecord`方法：

```jsx
  // 不能定义重复的路由信息，因为只会执行一次
  if (!pathMap[record.path]) {
    pathList.push(record.path)
    pathMap[record.path] = record
  }
```

可以看到还有另外一个数组对象，pathList ，这里用处就是保证根路由'/' 在数组中第一位。主要还是pathMap对象。

这里的pathMap对象就是扁平化之后的数据：

```jsx
pathMap = {
  '/': record,
  '/user/profile': record,
  '/user/posts': record,
  '/user/about': record,
  ...
  ...
}
```

其实还有一个nameMap，其原理跟pathMap一样，不断递归得到一个扁平化后的nameMap路由信息，只不过这时候对应的信息由原先path 映射 组件 换成了 name 映射组件。


到这里，由映射表创建已经完成，其实就是树的深度优先遍历的过程，最后将用户的路由配置扁平化成一个包含所有路由信息的对象。



  