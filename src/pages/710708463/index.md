---
title: '[JS练习题]-实现一个红绿灯，把一个圆形 div 按照绿色 3 秒，黄色 1 秒，红色 2 秒循环改变背景色'
date: '2020-09-29'
spoiler: ''
---

  最近再看winter老师的重学前端课程，其中一片文章[JavaScript执行（一）：Promise里的代码为什么比setTimeout先执行？](https://time.geekbang.org/column/article/82764)  （文章需要付费查看）阐述了宏任务与微任务之前的关系。

做下总结：**首先一个js脚本本身对于浏览器而言就是一个宏任务，也是第一个宏任务，而处于其中的代码可能有3种：非异步代码(console, Dom操作等)、产生微任务的异步代码（promise等）、产生宏任务的异步代码(settimeout、setinterval等)。
我们知道宏任务处于一个队列中，应当先执行完一个宏任务才会执行下一个宏任务，所以在js脚本中，会先执行非异步代码(比如开始的log语句)，再执行微任务代码(promise函数的执行)，最后执行宏任务代码。这时候我们进行到了下一个宏任务中，又按照这个顺序执行。处于同一级的情况下，微任务永远是宏任务的一部分，它处于一个大的宏任务内。**

文章结尾，布置了一道练习题，问：**实现一个红绿灯，把一个圆形 div 按照绿色 3 秒，黄色 1 秒，红色 2 秒循环改变背景色**。

下面是我的答案:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<style>
    #demo {
        width: 50px;
        height: 50px;
        border-radius: 25px;
    }
</style>
<body>
    <div id="demo"></div>

<script>

    let div = document.getElementById('demo')

    function sleep(wait) {
        return new Promise(function(resolve, reject) {
            setTimeout(resolve, wait)
        })
    }

    async function change() {
        while(1) {
            div.style.background = 'green'
            await sleep(3000)
            div.style.background = 'red'
            await sleep(2000)
            div.style.background = 'yellow'
            await sleep(1000)
        }
        
    }

    change()

</script>
</body>
</html>
```

  