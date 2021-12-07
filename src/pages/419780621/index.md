---
title: 'CSS 垂直, 水平居中常用方法'
date: '2019-03-12'
spoiler: ''
---

  # 垂直居中

## 效果图
![css布局](https://ws2.sinaimg.cn/large/006tKfTcly1g0zttmbqyoj308y07ejr9.jpg)
## 方法一：flex布局
```html
  <div class="container">
    <div class="flex-sub">
    </div>
  </div>
```
```css
  .container {
    width: 150px;
    height: 150px;
    border: 1px solid red;
    display: flex;
    align-items: center;
  }
  .container .flex-sub {
    width: 60px;
    height: 60px;
    background: aqua;
  }
```

## 方法二：绝对定位
```css
  .container {
    position: relative;
    width: 150px;
    height: 150px;
    border: 1px solid red;
  }
  .container .flex-sub {
    position: absolute;
    top: 50%;
    width: 60px;
    height: 60px;
    background: aqua;
    transform: translate(0, -50%);
  }
```
通过计算子元素height
```css
  .container {
    position: relative;
    width: 150px;
    height: 150px;
    border: 1px solid red;
  }
  .container .flex-sub {
    position: absolute;
    width: 60px;
    height: 60px;
    background: aqua;
    top: 50%;
    margin: -30px 0 0 0;
  }
```
## 方法三：margin: auto
```css 
 .container {
    position: relative;
    width: 150px;
    height: 150px;
    border: 1px solid red;
  }
  .container .flex-sub {
    position: absolute;
    width: 60px;
    height: 60px;
    background: aqua;
    top: 0;
    bottom: 0;
    margin: auto;
  }
```
## 方法四：padding
```css
  .container {
    width: 150px;
    padding: 50px 0;
    border: 1px solid red;
  }
  .container .flex-sub {
    width: 60px;
    height: 60px;
    background: aqua;
  }
```

# 水平垂直
## 效果图
![水平居中](https://ws4.sinaimg.cn/large/006tKfTcly1g0zyho9mj7j307k074a9x.jpg)

## 方法一：flex布局
```css
  .container {
    width: 150px;
    height: 150px;
    display: flex;
    justify-content: center;
    border: 1px solid red;
  }
  .container .flex-sub {
    width: 60px;
    height: 60px;
    background: aqua;
  }
```
## 方法二：绝对定位
```css
  .container {
    width: 150px;
    height: 150px;
    position: relative;
    border: 1px solid red;
  }
  .container .flex-sub {
    position: absolute;
    left: 50%;
    width: 60px;
    height: 60px;
    background: aqua;
    transform: translate(-50%, 0);
  }
```
通过计算子元素width
```css
  .container {
    width: 150px;
    height: 150px;
    position: relative;
    border: 1px solid red;
  }
  .container .flex-sub {
    position: absolute;
    width: 60px;
    height: 60px;
    background: aqua;
    left: 50%;
    margin: 0 0 0 -30px;
  }
```
## 方法三：margin：auto
```css
  .container {
    width: 150px;
    height: 150px;
    position: relative;
    border: 1px solid red;
  }
  .container .flex-sub {
    position: absolute;
    width: 60px;
    height: 60px;
    background: aqua;
    left: 0;
    right: 0;
    margin: auto;
  }
```
## 方法四：padding
```css
  .container {
    padding: 0 50px;
    height: 150px;
    border: 1px solid red;
  }
  .container .flex-sub {
    width: 60px;
    height: 60px;
    background: aqua;
  }
```
  