---
title: '适配iOS14桌面小组件'
date: '2020-09-27'
spoiler: ''
---

  随着iOS14正式版推送更新，想着蹭着热乎劲，把[Days](https://apps.apple.com/cn/app/days%E5%A4%87%E5%BF%98%E6%97%A5-%E7%BA%AA%E5%BF%B5%E9%87%8D%E8%A6%81%E7%9A%84%E6%97%A5%E5%AD%90/id1492204944)也适配一下。

经过2天左右的学习，看了WWDC有关桌面小组件的session。具体学习session顺序如下：

[小组件编程临摹课程1: 开始学习](https://developer.apple.com/videos/play/wwdc2020/10034)

[小组件编程临摹课程2: 变更时间线](https://developer.apple.com/videos/play/wwdc2020/10035)

[小组件编程临摹课程3: 加速时间线](https://developer.apple.com/videos/play/wwdc2020/10036)

具体的知识点以上session基本都讲了。这遍文章主要总结下适配过程以及遇到的坑。

目前主要适配了 .systemSmall  .systemMedium两种类型的桌面小组件，其UI布局是一样的。

<div>
 <img src="https://user-images.githubusercontent.com/15700015/94357145-c8f6aa80-00c8-11eb-8726-a65cca3d4654.png" width="20%" alt=""/>
 <img src="https://user-images.githubusercontent.com/15700015/94356322-af049a00-00bf-11eb-95d1-ec30b75fcd54.png" width="20%" alt=""/>
</div>

# 适配

项目采用的是Realm做的数据存储。在桌面小组件中如何共享App内的数据，这个跟Today组件是一样的。在新建的桌面小组件target中引入Realm。Podfile文件如下：

```ruby

def common_pods //公用
    pod 'RealmSwift'
end

target 'XXX' do
  common_pods
end

target "XXTodayWidget" do  //today组件
  common_pods
end

target "XXWidgetKit" do //桌面小组件
  common_pods
end
```

在`getSnapshot`和`getTimeline`方法中获取Realm存储路径

```swift
let directory = FileManager.default.containerURL(forSecurityApplicationGroupIdentifier: "你的App Group 名称")! as NSURL
var config = Realm.Configuration.defaultConfiguration
```

定义Model

```swift
class AddTimePlanModel: Object {
    @objc dynamic var title: String = "今天"
    ......
}
```

解析

```swift
let realm = try! Realm.init(configuration: config)
let results = realm.objects(AddTimePlanModel.self).sorted(byKeyPath: "order")
let model = results.first ?? AddTimePlanModel()
return model
```

布局UI

```swift
    var body: some View {
        let distance =  Date().timeStampToString().daysSinceNow(endDate: entry.model.startDate)
        ZStack {
            Image(entry.model.bgImgName)
                .resizable()
                .aspectRatio(contentMode: .fill)
                .mask(
                     RoundedRectangle(cornerRadius: 10)
                        .frame(minWidth: 0, maxWidth: .infinity)
                        .frame(minHeight: 0, maxHeight: .infinity)
                 )
                .frame(minWidth: 0, maxWidth: .infinity)
                .frame(minHeight: 0, maxHeight: .infinity)
                .background(Color.init(UIColor.init(hexFromString: entry.model.carBgColor)))
            
            VStack(alignment: .leading, spacing: 20) {
                Text(entry.model.title)
                    .foregroundColor(.white)
                    .font(.system(size: 22)).bold()
                
                HStack(alignment: .lastTextBaseline) {
                    Text("\(abs(distance))").foregroundColor(.white).font(.system(size: 40))
                    Text(distance >= 0 ? "剩余天数" : "累计天数").foregroundColor(.white).font(.system(size: 12))
                }
                
                Text(distance >= 0 ? "目标日: \(entry.model.startDate)" : "起始日: \(entry.model.startDate)")
                    .foregroundColor(.white)
                    .font(.system(size: 12))
           
            }
            
        }
    }
```

由于iOS14桌面小组件必须使用SwiftUI。在适配过程中，也了解下SwiftUI布局方式，后面单独写一篇介绍下。

# 遇到的问题

由于当时并不清楚了解到`IntentConfiguration `的具体含义：

```
`IntentConfiguration`,对于一个具有使用者可配置属性的小部件来说，您可以使用**SiriKit**自定义定义来定义属性。您使用SiriKit自定义定义来定义属性。例如，一个天气小部件需要一个城市的样式或邮政。 编码网址，或者一个包裹追踪Widget需要一个追踪号码。
```

就用了默认生成的`IntentConfiguration`，结果审核就给拒绝了。原因是审核人员不知道app哪些地方用到了SiriKit。我就意识到可能是`IntentConfiguration`。换成`StaticConfiguration`就过审了。`StaticConfiguration`类似于一个无状态的配置环境，`IntentConfiguration`需要主动去触发，提供一些显示内容设置项供用户选择。

```
StaticConfiguration：对于一个没有使用者可配置属性的小部件。“例如，显示一般市场资讯的股票市场小部件，或显示趋势标题的新闻小部件。
```

后续会为[Days](https://apps.apple.com/cn/app/days%E5%A4%87%E5%BF%98%E6%97%A5-%E7%BA%AA%E5%BF%B5%E9%87%8D%E8%A6%81%E7%9A%84%E6%97%A5%E5%AD%90/id1492204944)提供桌面小组件内容显示配置项，比如可以选择哪一个纪念日，纪念日的分类等等，网格状纪念日等等。


  