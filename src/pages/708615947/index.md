---
title: '【JS基本功系列】执行环境，作用域链(scope chain)'
date: '2020-09-25'
spoiler: ''
---

  ### 执行环境与作用域链的关系

在JS中，每一个执行环境都有一个相关的变量对象，这个变量对象中包含了在这个环境中定义的所有变量和函数，在Web浏览器中，全局执行环境被认为是window对象，window就拥有了全局环境中所有元素的访问权。

例如我们经常会申明一个全局变量来保存某项数值的变化：

```jsx
var sum = 0
function add() {
    sum += 10
    return sum
}
console.log(sum) //0
console.log(add()) //10

console.log(window.sum) //10
console.log(window.add()) //20
```

虽然前两个输出语句省略了window对象，但是浏览器执行js解析器时，会默认使用window对象。后两句输入语句我们显示的加上了window对象，则输出为`sum`变量叠加后的数值。

某个执行环境中所有的代码执行完成后，该环境被销毁掉，随之其中所有的变量和函数都会销毁，全局执行环境知道应用程序退出，比如关闭浏览器或者网页。

例如下面代码：

```jsx
var sum = 0
function add2() {
    var temp = 10
    sum = sum + temp
    return sum
}

console.log(sum) // 0

console.log(add2()) // 10
console.log(temp) // temp is not defined
```

当`add2()`函数执行完时，其内部变量`temp`就被销毁了，函数外部就不能再访问了。

每个函数都有自己的执行环境，当代码在这个环境中执行，就会创建变量对象的**作用域链**，他保证了对执行环境所有变量和函数的顺序访问，顺序访问是什么，下面的列子可以解释：

```jsx
var scope = 100
function getLevel() {
    if (scope == 100) {
        return 'A+'
    } else {
        return 'A-'
    }
}

console.log(getLevel()) // 'A+'
```

在评选等级函数中，用到了`scope == 100`的判断条件，当程序执行到这一步时，`getLevel`执行环境会访问`scope`变量，`scope`变量并没有在它的执行环境中申明，程序就会去它的下一个执行环境中寻找，它的下一个执行环境就是全局执行环境，在全局执行环境中，window对象有一个属性为`scope`，那么这里访问的就是window的`scope`属性。

这个列子就更能体现出作用域链的作用。

```jsx
var scope = 100
function getLevel() {
    var base = 80
    function getBase() {
        var isPass 
        if (scope >= base) { //在当前执行环境中，可以通过作用域链访问到外部执行环境的变量，即可以访问scope base 和当前环境中的isPass
            isPass = true
        } else {
            isPass = false
        }
        return isPass
    }
    return getBase()
}

console.log(getLevel()) // true
```

上面这个例子一共3个执行环境。从内部到最外部依次为：`getBase()`执行环境，其内部包含了一个`isPass`变量; `getLevel`执行环境，其内部包含了一个`base`变量;  最外部是全局环境。在最内部执行环境`getBase()`中，可以访问`isPass` `base` `scope`变量，`getLevel()`执行环境中可以访问`base` `scope`变量，不能访问`isPass`。 全局环境就只能访问`scope`变量。

![编组@3x](https://user-images.githubusercontent.com/15700015/94223940-c3ac2b00-ff23-11ea-9f77-1b88158c952c.png)


总结： 内部执行环境可以通过作用域链访问所有的外部执行环境，但外部执行环境不能访问内部执行环境中的任何变量和函数。这些环境之间的联系是有序的。每个环境都可以向上搜索作用域链，以查询变量和函数名；但任何环境都不能通过向下搜索作用域链而进入另个执行环境。

  