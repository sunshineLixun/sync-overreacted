---
title: 'React更新props与Echarts实际应用'
date: '2018-11-28'
spoiler: ''
---

  # 关于Echarts初始化

在react项目中引用Echarts组件，需要在 `render()`方法中返回一个`div`元素。因为Echarts底层使用Canvas绘图，所以`div`元素的宽高一定是已知的。列如：

.js

```jsx
  render() {
    return (
      <div id="pieCharts" className={styles.pieCharts}>
      </div>
    )
  }
  
  componentDidMount() {
    let myChart = echarts.init(document.getElementById('pieCharts'))
    let options = this.setOptions()
    myChart.setOption(options)
  }
  
  setOptions() {
    const {take_off, spray_off, spray_on, hand, total} = this.props

    /*
      该方法中拿到父组件传递过来的props之后，开始巴拉巴拉按照echarts官方文档设置需要的参数。
    */
  }
```

.less

```
.pieCharts {
  width: 400px;
  height: 250px;
  margin: auto;
}
```

# 动态更新Echarts数据参数

如果该组件只是静态去展示某个时间段的数据，完全满足需求。但是如果是动态的去生成数据时，也就是props每时每刻都在改变，那么会发现echarts所展示的数据根本没有发现变化。我们都知道`componentDidMount`方法只会执行一次，就是在组件挂载完成之后调用。解决该问题就需要用到更新props或者state那一套流程。(React生命周期参考React官网)

```jsx
  shouldComponentUpdate(nextProps) {
    if (nextProps.take_off !== this.take_off ||
      nextProps.spray_off !== this.spray_off ||
      nextProps.spray_on !== this.spray_on ||
      nextProps.hand !== this.hand) {
      return true
    }
    return false
  }
  
  componentDidUpdate() {
    const dom = document.getElementById('pieCharts')
    var myChart = echarts.getInstanceById(dom.getAttribute('_echarts_instance_'));
    myChart.setOption(this.setOptions())
  }
```

`shouldComponentUpdate`判断当前组件是否需要更新，一般多用于性能优化，减少不必要的组件刷新次数。在本例中，比较父组件传递过来的props与上一次props的异同。当返回true时，我们在`componentDidUpdate`方法中取到echarts的实例，重新对echarts设置数据参数。

# 小结

需求不难实现，最主要的还是要对React的生命周期有详细的认知。

![React生命周期](https://raw.githubusercontent.com/sunshineLixun/bloggerImage/master/imagesreact_life_cycle.png)

# 参考

[ReactJs component lifecycle methods — A deep dive](https://hackernoon.com/reactjs-component-lifecycle-methods-a-deep-dive-38275d9d13c0)
[如何通过dom对象获取echarts对象](https://github.com/apache/incubator-echarts/issues/1400)
  