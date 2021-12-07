---
title: '近期面试总结'
date: '2020-10-16'
spoiler: ''
---

  # 感悟
最近2天陆续面了几家，每个公司的侧重点不同。有的注重基础知识的和遇到实际问题解决办法的考察，有的根据当前公司技术栈与应聘者技术栈符合度考察。有的则是以工作经历简历上的内容牵引出知识点作为考察。以下是对面试题做个分类，从基础知识到实际开发遇到问题解决办法。

## 基础知识

> 1.`script标签中async defer区别是什么？`

`<script src="script.js"></script>`
没有 defer 或 async，浏览器会立即加载并执行指定的脚本，“立即”指的是在渲染该 script 标签之下的文档元素之前，也就是说不等待后续载入的文档元素，读到就加载并执行。会阻塞当前文档渲染进度。

`<script async src="script.js"></script>`
有 async，加载和渲染后续文档元素的过程将和 script.js 的加载与执行并行进行（异步）。不会根据文档解析是否完成，只要在异步队列中完成script.js的加载之后，就会立即执行。

`<script defer src="myscript.js"></script>
`有 defer，加载后续文档元素的过程将和 script.js 的加载并行进行（异步），但是 script.js 的执行要在所有元素解析完成之后，DOMContentLoaded 事件触发之前完成。

![284aec5bb7f16b3ef4e7482110c5ddbb_articlex](https://user-images.githubusercontent.com/15700015/96208022-2f236e80-0f9f-11eb-8ec4-9319ebc05cf8.jpeg)

> 2. white-space中pre有什么作用 

```html
<html>
<head>
<style type="text/css">
p {white-space: pre;}
</style>
</head>
<body>

<p>This     paragraph has    many
    spaces           in it.</p>
<p>注释：当 white-space 属性设置为 pre 时，浏览器不会合并空白符，也不会忽略换行符。</p>
</body>
</html>
```
效果:

<img width="530"  alt="截屏2020-10-16 11 08 01" src="https://user-images.githubusercontent.com/15700015/96208345-dd2f1880-0f9f-11eb-896a-93be01207ee3.png">

> 3 对象深度copy有哪些方法

浅拷贝: 仅仅是复制了引用地址，换句话说，复制了之后，原来的变量和新的变量指向同一块内存地址空间，彼此之间的操作会互相影响。
深拷贝：而如果是在堆中重新分配内存，拥有不同的地址，但是值是一样的，复制后的对象与原来的对象是完全隔离，互不影响，为 深拷贝。

深度copy实现方法：

1.对象序列化，JSON.strigify 和 JSON.parse

2：递归。递归的原理：如果对象包含有引用类型时，要将引用类型中值类型属性找出来进行复制copy。只有对象中所有值类型完成copy才算整个对象完成深度copy。（值类型的copy互不影响，属于深度copy）。

```jsx
export function deepCopy(target) {
  let res = null
  if (typeof target == 'function') {
    res = []
    target.forEach(item => {
      res.push(deepCopy(item))
    })
  }
  if (typeof target == 'object') {
    res = {}
    Object.keys(target).forEach(key => {
      res[key] = deepCopy(target[key])
    })
  }
  return res || target
}

```

> 4.Map与Object的区别？Map与WeakMap的区别？

| 对比项 | 映射对象Map | Object对象 |
| :---- | :----: | :----: |
| 存储键值对 | ✔️ | ✔️ |
| 遍历所有键值对 | ✔️ | ✔️ |
| 检查是否包含指定的键值对 | ✔️ | ✔️ |
| 使用字符串作为键 | ✔️ | ✔️ |
| 使用Symbol作为键 | ✔️ | ✔️ |
| 使用任意对象作为键 | ✔️ |  |
| 可以很方便的得知键值对的数量 | ✔️ |  |

Map与WeakMap的区别:

- 键值对：Map可以使用任何javascript数据类型作为键，WeakMap中的键只能是Object或者继承自Object的类型，使用非对象设置键会抛出TypeError。值的类型则没有限制。

```jsx
const vm = new WeakMap();
vm.set("obj", "1")  //Uncaught TypeError: Invalid value used as weak map key
```

```jsx
const map = new Map()
map.set("obj", "1")
console.log(map.get("obj")) // 1
```

- 垃圾回收机制：WeakMap中如果键值是空对象，这个对象如果没有被其他类型引用，这个对象键就会被当作垃圾回收，键/值对就会从弱映射中消失，成为一个空映射，键值对遭到破坏后，值本身也会成为垃圾回收的目标。

- WeakMap是不可枚举的，无法获取大小。因为WeakMap中的键值对任何时候都可能会被销毁。

```jsx
const vm = new WeakMap();
let key = {
    a: 1
}
vm.set(key, "1")
for (const iterator of vm) {
    console.log(iterator) // TypeError: vm is not iterable
}
```

> 5. for in  与 for of  的区别
for in 用于枚举对象中非符号键属性，for of 用于枚举可迭代对象的元素

```jsx
function Person() {  }
Person.prototype.name = "lixun"
Person.prototype.hegiht = 178
const person = new Person()
for (const key in person) {
    console.log(key) //name height
}

for (const el of [1, 4, 5, 7]) {
    console.log(el) // 1  4  5  7
}
```

## 进阶

> 如何实现订阅-发布模式

概念: 

观察者模式定义了对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知，并自动更新。观察者模式属于行为型模式，行为型模式关注的是对象之间的通讯，观察者模式就是观察者和被观察者之间的通讯。

观察者模式有一个别名叫“发布-订阅模式”，或者说是“订阅-发布模式”，订阅者和订阅目标是联系在一起的，当订阅目标发生改变时，逐个通知订阅者。我们可以用报纸期刊的订阅来形象的说明，当你订阅了一份报纸，每天都会有一份最新的报纸送到你手上，有多少人订阅报纸，报社就会发多少份报纸，报社和订报纸的客户就是上面文章开头所说的“一对多”的依赖关系。另外一个例子：你在微博上关注了A，同时其他很多人也关注了A，那么当A发布动态的时候，微博就会为你们推送这条动态。A就是发布者，你是订阅者，微博就是调度中心，你和A是没有直接的消息往来的，全是通过微博来协调的（你的关注，A的发布动态）。

在OC/Swift中，`通知(NSNotification)`是一种订阅-发布模式:

```swift
//发送通知
NotificationCenter.default.post(name: NSNotification.Name(rawValue: "viewNotiSecond"), object: nil, userInfo: nil)

//监听通知
NotificationCenter.default.addObserver(self, selector: #selector(receive(noti:)), name: NSNotification.Name("viewNotiSecond"), object: nil)
```

在js语言中如何实现?

```jsx
class Publish {
    constructor() {
        this.subscribers = [] //缓存订阅者
    }

    subscribe(topic, callback) { //topic为订阅者唯一标识符
        let callbacks = this.subscribers[topic]; //获取订阅者
        if(!callbacks) {
            this.subscribers[topic] = [callback]; //有订阅者，讲订阅者放到指定数组索引中
        } else{
            callbacks.push(callback); //存在，缓存到数组中
        }
    }

    publish(topic, ...args) {
        let callbacks = this.subscribers[topic] || []; //根据标识符找到订阅者
        callbacks.forEach(callback => callback(...args)); //遍历订阅者，执行回掉
    }

}

// 创建事件调度中心，为订阅者和发布者提供调度服务
let pubSub = new Publish();
pubSub.subscribe('WeiBo', console.log);
pubSub.subscribe('WeiBo', console.log);
pubSub.publish('WeiBo', '我发了一条微博信息');
```

在终端中执行`node subscribr.js`得到输出:
```
➜  订阅-发布 node subscribe.js
我发了一条微博信息
我发了一条微博信息
```
参考链接：https://www.cnblogs.com/onepixel/p/10806891.html

  