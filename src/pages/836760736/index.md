---
title: 'rollup打包使用'
date: '2021-03-20'
spoiler: ''
---

  ### 背景
最近把公司小程序项目写的一些公共组件打算提取出来放到npm上，以后每个项目都直接引用这个npm包就行。由于采用的技术栈是Taro，所以就参考了下taro-ui的打包方式。尽量做到统一性。
公共组件打包上传到npm自然绕不开webpack、[rollup](https://rollupjs.org/guide/en/)这种打包方式。简单说下他们的区别吧。

- webpack：适合用于做一些脚手架项目相关的，脚手架牵扯到的第三方库比较多。
- rollup打出的包体积要比webpack更小，因为webpack打出的包里面含有很多__webpack_require__它自定义的一些工具函数。 对于公共组件库来说，体积越小越好。
- 在打包后的代码可读性来说，rollup打包后代码可读性更好。没有那些webpack里面一堆__webpack_require__函数

暂时就知道这么多，欢迎补充。

### 使用

新建项目 `rollup-build-demo`，cd 到项目目录下执行 `npm init`。然后引入rollup，`yarn add rollup -D`

![1616229292(1)](https://user-images.githubusercontent.com/15700015/111864137-3f020d00-899a-11eb-8bfe-7a83289cac0f.jpg)

我们可以根据命令来打包。
针对浏览器执行`rollup main.js --file bundle.js --format iife` 
这里有几个参数解释下： main.js是入口文件，bundle.js是打包后的js文件，format iife 表示iife格式。输出文件格式一共有五种: `amd /  es6 / iife / umd / cjs`
这几种格式就是js模块划。

一般的对于项目来说肯定要配置打包文件比较好。rollup的默认配置打包文件是`rollup.config.js`。
在项目根目录下新建`rollup.config.js`。配置如下

![1616229753(1)](https://user-images.githubusercontent.com/15700015/111864363-61485a80-899b-11eb-8227-e2bd2ce17191.jpg)

新建src目录，新建index.js文件作为输入文件。

### 配置说明
#### 1.input
入口文件地址
#### 2.output
```jsx
output:{
    file:'bundle.js', // 输出文件
    format: 'cjs,  //  五种输出格式：amd /  es6 / iife / umd / cjs
    name:'A',  //当format为iife和umd时必须提供，将作为全局变量挂在window(浏览器环境)下：window.A=...
    sourcemap:true  //生成bundle.map.js文件，方便调试
}
```
#### 3.plugins
各种插件：babel、css、node、json、等等
#### 4.external
```
external:['taro', 'taro-ui', 'react'] //告诉rollup不要将这些第三方库打包，而作为外部依赖
```
#### 5.global
```jsx
global:{
    'jquery':'$' //告诉rollup 全局变量$即是jquery
}
```
### 编写脚本，打包输出
在package.json文件中编写打包脚本。根据rollup的[Command line](https://rollupjs.org/guide/en/#command-line-flags)打包命令是 `rollup -c`。

![1616230330(1)](https://user-images.githubusercontent.com/15700015/111864571-a5882a80-899c-11eb-843d-38bb05b59837.jpg)

在src的index.js编写测试代码

![1616230470(1)](https://user-images.githubusercontent.com/15700015/111864604-f7c94b80-899c-11eb-8a11-7c603721889f.jpg)

执行 `yarn build`之后，去dist文件下看输入文件

![1616230538](https://user-images.githubusercontent.com/15700015/111864626-1f201880-899d-11eb-9968-657416328b67.jpg)

可以看到命令行中打印：
```shell
$ yarn build
yarn run v1.22.5
$ rollup -c

src/index.js → dist/index.js...
created dist/index.js in 24ms
Done in 0.38s.
```
src下的入口文件被打包到dist目录下的index.js。

# 使用babel

可以看到打包后的还是es6代码，对于一些低版本浏览器，或者领导让你兼容IE。哈哈，这还不行。那我们的转换成es5代码。
引入babel。
`yarn add rollup-plugin-babel @babel/core @babel/preset-env -D`
在根目录下新建.babelrc文件，配置babel。
```
{
    "presets": [
        [
            "@babel/env",
            {
                "modules": false
            }
        ]
    ]
}
```
然后在rollup.config.js文件中，使用babel插件

![1616230977(1)](https://user-images.githubusercontent.com/15700015/111864822-34e20d80-899e-11eb-9b67-7b2d11c79ac4.jpg)

再执行`yarn build`。再看打包之后的代码。

![1616231094](https://user-images.githubusercontent.com/15700015/111864860-6eb31400-899e-11eb-850f-84853623a9bb.jpg)

可以看到已经转换成es5代码了。

# 剔除外部第三方库，减少dist包体积

我们在开发中避免不了使用第三方插件库，如果将它们一块build得到dist包中，所有 imports 的模块打包在一起 ，势必会增大包体积。我们可以使用`externals`剔除外部包。

入口文件 index.js 引入lodash

![1617271511(1)](https://user-images.githubusercontent.com/15700015/113280742-4a98f080-9317-11eb-8578-1521d2464eea.jpg)

执行`yarn build`

![1617272448(1)](https://user-images.githubusercontent.com/15700015/113280602-13c2da80-9317-11eb-9cd7-83c52a7a4fc2.jpg)

可以看到lodash的源码也被打包进来了。

使用`externals`剔除

![1617272643(1)](https://user-images.githubusercontent.com/15700015/113280913-8338ca00-9317-11eb-9c35-8be78c2ded16.jpg)

执行`yarn build`之后再来看看dist的输出文件

![1617272629(1)](https://user-images.githubusercontent.com/15700015/113280963-9186e600-9317-11eb-8477-5cdc72df091f.jpg)

可以看到lodash作为require被引入进来了。 同理我们在实际开发中也可以将react、vue作为外部依赖require打包输出文件中，大大减少了build包体积。

这里说下`@rollup/plugin-node-resolve` 和 `@rollup/plugin-commonjs`插件
rollup.js编译源码中的模块引用默认只支持 ES6+的模块方式import/export。然而大量的npm模块是基于CommonJS模块方式，这就导致了大量 npm 模块不能直接编译使用。所以辅助rollup.js编译支持 npm模块和CommonJS模块方式的插件就应运而生。

@rollup-plugin-node-resolve 插件允许我们加载第三方模块 @rollup/plugin-commons 插件将它们转换为ES6版本

未完待续...


  