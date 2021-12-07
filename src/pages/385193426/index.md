---
title: 'React Native 自定义组件'
date: '2018-11-28'
spoiler: ''
---

  # 方法一：

自定义组件(Button)代码：

```jsx
export default class Button extends Component{
    render(){
        //解构
        const {title,onClick} = this.props;
        return(
            <TouchableOpacity style={styles.touchStyle}
                              onPress={onClick}
            >
                <Text style={styles.textStyle}>{this.props.title}</Text>
            </TouchableOpacity>
        )
    };
}
```

`export default`关键字：导出自定义组件,可以被别的js文件所引用。如果没有关键字,就只能在本js文件使用该类。例如在`Login.js`文件中可以用

```
import Button from './Button'
```
导入Button组件

```jsx
//解构
const {title,onClick} = this.props;
```

解构从父组件的传过来的props, 子组件拿到父组件设置的属性值,显示传递的值或者处理传递过来的方法。

列如上面代码:

```jsx
///显示Button的title
<Text style={styles.textStyle}>{this.props.title}>
</Text>

///处理TouchableOpacity的点击事件
<TouchableOpacity style={styles.touchStyle}
                  onPress={onClick}>
</TouchableOpacity>                  
```

父组件调用代码: 传递title信息和onClick方法回调

```jsx
<Button title="登录"
        onClick={() => alert('点击了登录按钮')}/>    
```
# 方法二

与方法一的区别不大

代码：

```jsx
import React, { Component,PropTypes } from 'react';
import { Text,TouchableOpacity,StyleSheet } from 'react-native';

export default class Button extends Component{
    static propTypes = {
        title: PropTypes.string,
        onPress: PropTypes.func
    }

    render(){
        return(
            <TouchableOpacity style={styles.touchStyle}
                              onPress={()=>{
                                if (this.props.onPress) {
                                    this.props.onPress(this)
                                }
                              }}
            >
                <Text style={styles.textStyle}>{this.props.title}</Text>
            </TouchableOpacity>
        )
    };
}
```

其中

```jsx
static propTypes = {
        title: PropTypes.string,
        onPress: PropTypes.func
    }
```
暴露出组件的属性(title)和方法(onPress)。

```jsx
<TouchableOpacity style={styles.touchStyle}
                              onPress={()=>{
                                if (this.props.onPress) {
                                    this.props.onPress(this)
                                }
                              }}
            >
                <Text style={styles.textStyle}>{this.props.title}</Text>
            </TouchableOpacity>
```

- 处理TouchableOpacity的点击事件,设置回调函数
- 显示Button的title

组件调用：

```
<Button  title="登录"
        onPress={() => alert('点击了登录按钮')}/> 
```

# Demo

![RNCustomComponent.gif](https://i.loli.net/2018/03/02/5a98efa4c7dc0.gif)
  