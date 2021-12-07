---
title: 'AutoLayout - 父视图根据子视图布局自适应大小'
date: '2018-11-28'
spoiler: ''
---

  # 前言
本文记录AutoLayout日常布局UI时的技巧。每当拿到新的界面设计时,比

如这样的: 
	
<img src="/images/1.png" width=50% height=50% align=center/>

当某个字段的长度不确定时,(图1中的“作业地址”,图2中"预约地址"和“取消原因”)。Label的行数就要设置为0或者 >=1。总试图的高度就不确定。于是乎就要开始就算containerView的高度了。


```obj-c
- (CGFloat)cellHeight{
	CGFloat publicHeight = kViewTopPadding + kImageSize().height + kViewTypeMargin + _nameStrSize.height + kViewTypePadding + _nameTextSize.height + kViewTypeMargin / 2 + kCellBottomPadding;
	switch (_layoutViewType) {
		case ViewLayoutBreviaryType:
			return publicHeight;
			break;
		case ViewLayoutDetailType:
			//判断当前model存储的信息是待执行还是已取消状态
			if (_orderModel.reserveType == YWBOrderReservationToBeConfirm) {
				return publicHeight + kViewTypeMargin / 2 + _addressStrSize.height + kViewTypePadding + _addressTextSize.height + kViewTypeMargin + _typeStrSize.height + _typeTextSize.height + kViewTypeMargin + kButtonHeight + kViewBottomPadding + kCellBottomPadding;
			}else if(_orderModel.reserveType == YWBOrderReservationCancledForSaler || _orderModel.reserveType == YWBOrderReservationCancledForClient){
				return publicHeight + kViewTypeMargin / 2 + _addressStrSize.height + kViewTypePadding + _addressTextSize.height + kViewTypeMargin + _cancleStrSize.height + kViewTypePadding + _cancleTextSize.height + kViewBottomPadding + kCellBottomPadding;
			}
		   break;
	}
	return 0.0;
}

```

```obj-c
- (CGFloat)heightWithWidth:(CGFloat)width font:(UIFont *)font
{
	CGRect attrsRect = [self boundingRectWithSize:CGSizeMake(width, CGFLOAT_MAX)
										  options:NSStringDrawingUsesLineFragmentOrigin | NSStringDrawingUsesFontLeading
									   attributes:@{NSFontAttributeName : font}
										  context:nil];
	return attrsRect.size.height;
}

```

如果不细心漏掉某一处高度,查找起来也比较麻烦。每次算高度是我不爱干的一件事情。没有技术含量😂。完全就是细心活。多亏AutoLayout能让我们从这一毫无兴趣的工作中解放出来。

# 父视图自适应

新建Xcode工程。在Main.storyboard的ViewController中随意拖入view控件，颜色暂定为blue。放置3个Label控件。

![1.png](https://i.loli.net/2018/03/02/5a98ee340eaab.png)

1. 先不设置blueview的约束。
2. 逐步设置Label的约束。

黑色Label约束如下:

![2.png](https://i.loli.net/2018/03/02/5a98ee7ee44ec.png)

在这里我设置了黑色Label的size。这个并无关系，可设可不设。使用Label的固有大小也行。

黄色Label约束如下:

![3.png](https://i.loli.net/2018/03/02/5a98eea02e37a.png)

红色Label约束如下:

![4.png](https://i.loli.net/2018/03/02/5a98eeb892428.png)

Label的行数都为0。

## 重点
***Label一定要设置距离父视图的间距。这样AutoLayout才能计算出父视图的大小。***

# 最重要的一步

看下view的约束
![5.png](https://i.loli.net/2018/03/02/5a98eed285ff8.png)
![6.png](https://i.loli.net/2018/03/02/5a98eee554feb.png)
将view的固有尺寸设置为placeholder。 我们都知道Label Button ImageView(已经设置image属性)这些控件，在用AutoLayout布局时，只需要设置外边距就行了。AutoLayout会自动适应size大小。这就是固有尺寸。反观View就不行了。假如只设置View的外边距，这时一定会报出约束错误。但是View作为容器。它能根据子控件的大小来自适应。只需要将其固有尺寸(size)都设置为0。

这是对于高度不固定的来讲。如果宽度不固定呢？答案是同样适用，一般来说，view的宽度是固定的。高度不固定。

## 需要注意的问题
如果不设置view的左右边间距或者宽度你就会发现变成了这样:
![7.png](https://i.loli.net/2018/03/02/5a98eefb35dd4.png)
view的左右边距已经被Label撑到屏幕外面了。因为这时view的宽度也是会随着Label变化的。
所以说我设置了view的左右边距(或者设置其固定的宽度)

## 验证
接来下就可以设置Label的text属性了。随便输入汉字来验证一下：
![8.png](https://i.loli.net/2018/03/02/5a98ef188c27f.png)
![9.png](https://i.loli.net/2018/03/02/5a98ef188f224.png)

可以发现view的高度已经随着Label的变化而变化了。其他的控件以此类推。都是一样的道理。另外纯代码布局也同样适用。

# 总结
view作为容器类其承载其他控件的显示。只要设置了子控件的距离自己的约束。并将自己的固有尺寸设置为placeholder(也就是width和height都为0)。这样AutoLayout就能自动计算出view的大小。

# 补充
怎么样拿到view的大小呢？系统为我们提供了这样的API

```swift
extension UIView {

    /* The size fitting most closely to targetSize in which the receiver's subtree can be laid out while optimally satisfying the constraints. If you want the smallest possible size, pass UILayoutFittingCompressedSize; for the largest possible size, pass UILayoutFittingExpandedSize.
     Also see the comment for UILayoutPriorityFittingSizeLevel.
     */
    @available(iOS 6.0, *)
    open func systemLayoutSizeFitting(_ targetSize: CGSize) -> CGSize // Equivalent to sending -systemLayoutSizeFittingSize:withHorizontalFittingPriority:verticalFittingPriority: with UILayoutPriorityFittingSizeLevel for both priorities.

    @available(iOS 8.0, *)
    open func systemLayoutSizeFitting(_ targetSize: CGSize, withHorizontalFittingPriority horizontalFittingPriority: UILayoutPriority, verticalFittingPriority: UILayoutPriority) -> CGSize
}
```

在viewDidLayoutSubviews或者layoutSubviews中通过调用上面API就能够获得通过AutoLayout计算过后view的大小。
  