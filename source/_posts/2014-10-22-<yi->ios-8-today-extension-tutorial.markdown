---
layout: post
title: "<译>iOS 8 Today Extension Tutorial"
date: 2014-10-22 23:41:28 +0800
comments: true
categories: 
---

**Ray提醒: ** 这篇文章可以说是一个精简版，它是从[iOS8 by Tutirials](http://www.raywenderlich.com/store/swift-tutorials-bundle)这本书中摘录的一个章节。[iOS8 by Tutirials](http://www.raywenderlich.com/store/swift-tutorials-bundle)是我们[iOS8 Feast](http://www.raywenderlich.com/82230/introducing-ios-8-feast)套餐中的一部分。通过这篇文章我们可以看到书中的内容是怎么样的。希望你喜欢！

iOS8介绍了一种新的概念 `App Extensions`:一种在操作系统上与其他应用共享应用程序功能的方式！

这些`Extensions`中有一种叫`Today Extensions`，又称`Widgets`。它可以让你在`Notification Center`中呈现信息。这是一种很好的方式向用户直接的提供他感兴趣的最新的信息。

在这个教程中，你会编写一个`Today Extension`用来渲染[bitcoin.org](https://bitcoin.org/en/)中比特币基于美元的当前市场价格。

##介绍比特币
如果你对比特币还不熟悉，那用一句很短的话来介绍的话就是比特币是一种还处于初期的数字加密货币。除了可以用来作P2P的交换和购买外，比特币交易还允许用户将比特币交易成另外一种数字加密货币，比如狗狗币和莱特币，以及像美金和欧元这样的常用货币。

##介绍Crypticker项目
一旦你开始写一个`Extension`，你必须先需要一个主App来做扩展；看看`Crypticker`项目吧。

`Crypticker`是用来展示当前的比特币价格、当前价格与昨日价格的区别和历史价格图表的一个简单的App。图表包括30天的历史价格；滑动你的手指可以看到过去特定的某天的准确价格。

`Extension`会包含所有这些功能，除了点击图表可以看到特定某天的价格。`Today Extension`也有局限性，尤其是当它在滑动(`swipe`)的时候，`Swipe Gesture`总是在`Notification Center`中的`Today`和`Notifications`这两个`sections`里被触发，所以他没有真的做到提供了最好的或者最可靠的用户体验。

## 入门指南

我们从下载[Crypticker起始项目](http://cdn4.raywenderlich.com/wp-content/uploads/2014/09/crypticker-swift-starter-GM.zip)入手。这个项目包含整个如上面描述的`Crypticker` app。我们的教程不会关注这个项目本身的开发，所以你愉快又惊讶的发现这个教程是如此的简洁。毕竟，你要写一个`Extension`，而不是整个app。
编译运行。请注意你需要一个可以工作的网络来链接`web service`以及拉取实时的价格。

(译者注：这个App还是Swift1.0的代码，可以根据Xcode的提示将代码修改成符合Swift1.1，主要是Swift1.1加入了可失败初始化功能)

<p align="center">
	<img style="width:281px;height:500px;" src="/images/custom/post/ios-8-today-extension-tutorial/CryptickerApp-1.png" >
</p>
<br/>

App看起来非常像上面的屏幕截图；当然显示的数据是基于当前的比特币市场怎么样。触碰在底部附近的图表会绘制一条线以及显示遮天相关的价格。

## BTC Widget
为了不熟悉的人更好的理解，BTC就是比特币的缩写。很像USD就代表美元。`Today Extension`会渲染出一个`Crypticker`主视图的缩小版本。

理论上，`Crypticker`有能力展示多种加密货币的价格，但是你的`Extension`是对BTC的特制版。因此，它的名字应该叫`BTC Widget`。

```
Note: Extensions,就其本质而言，只有一个目的。如果你想要提供出你另一个数字加密货币的信息，比如狗狗币（Dogecoin），最好的方法是针对你的App打第二个widget的包或者适当的设计你的UI，也许就像股票Widget。
```

在这篇教程的结尾，你的`Today Extension`会看起来像这样:

<p align="center">
	<img style="width:281px;height:500px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-Widget-2.png" >
</p>
<br/>

### 添加一个Today Extension target

所有的`Extention`被包装成一个和他们主工程隔离的`binary`。所以你需要为你的`Crypticker`工程添加一个`Today Extention`的`target`。

在Xcode的`Project Navigator`中，选中`Crypticker project`，然后选择`Editor\Add Target…`来添加一个新的`target`。当模板选择器出现后，选择`iOS\ Application Extension`下的`Today Extension`。点击`Next`。

<p align="center">
	<img style="width:281px;height:500px;" src="/images/custom/post/ios-8-today-extension-tutorial/AddTarget-3-700x411.png" >
</p>
<br/>

设置`Product Name`为`BTC Widget`，然后验证`Language`为`Swift`，`Project`为`Crypticker`，`Embed in Application`也是`Crypticker`。点击`Finish`。


<p align="center">
	<img style="width:281px;height:500px;" src="/images/custom/post/ios-8-today-extension-tutorial/AddTarget-4-700x413.png" >
</p>
<br/>

当提示激活`BTC Widget``scheme`的时候.就像文本指示器一样，会为你创建另一个`Xcode``scheme`。

恭喜！现在`BTC Widget`会展示在你的`targets`的列表中。


<p align="center">
	<img style="width:281px;height:500px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-AddTarget-5-250x250.png" >
</p>
<br/>

确保你选中了`BTC Widget`target，选择`General`标签，然后按下在`Linked Frameworks and Libraries`下的`+`按钮。

<p align="center">
	<img style="width:281px;height:500px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-LinkFramework-6-480x294.png" >
</p>
<br/>

选中`CryptoCurrencyKit.framework`然后点击`Add`。

CryptoCurrencyKit是一个在`Crypticker app`中使用的用来从网络上获取货币价格的自定义的`framework`。你很幸运，有个难以置信般善良和体贴的程序猿模块化了`Crypticker`的代码到`framework`，所以它可以在多个`target`下共享 :]

为了在主工程和它的extensions质检共享代码，你必须使用自定义`framework`。如果你不这么做，你会发现你重复撰写了很多代码以及违反了软件工程中一个很重要的原则：`DRY- or，Dont Repeat Yourself`，我再说一次：`Dont Repeat Yourself`（不要重复发明轮子）

现在，你已经准备好开始实现`extension`了。

注意，现在有一个以你新的`target`命名的文件组在你项目的`Navigator`中，`BTC Widget`。这是`Today Extension`代码默认分组存放的地方。

展开这个文件组，你会看到这里有一个`view controller`，一个`storyboard`文件和一个`Info.plist`文件。它的`target`配置信息也告诉他去`MainInterface.storyboard`加载他的`interface`，`MainInterface.storyboard`包含一个指定class为 `TodayViewController.swift`的 `ViewController`。

<p align="center">
	<img style="width:281px;height:500px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-ListofFiles-7-274x320.png" >
</p>
<br/>

你会发现有一些你期望看到的文件在`Today Extension`模板中却消失不见了。比如说`app delegate`。记住所有的Extension都运行在另外一个主工程的内部，所以他们和传统的app的生命周期不一样。

本质上，Extension的生命周期是被映射在`TodayViewController`的生命周期上的。

打开`MainInterface.storyboard`。你会看到一个带着明亮的`Hello World``label`的深色的`View`。为了与通知中心的暗色调协调，当`Today Extension`有一个清晰透明的背景和一个明亮或者艳丽的文本时是最清晰易读的。

确保`BTC Widget`scheme在`Xcode`的`toolbar`中被选中，然后编译运行。这时会出现一个窗口来问你哪一个app需要运行。Xcode正在问你哪个主应用需要运行。选择`Today`。这会告诉你的iOS打开通知中心的`Today`视图。它会依次展开你的`Widget`。

<p align="center">
	<img style="width:281px;height:500px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-PickAppToRun-8-349x320.png" >
</p>
<br/>

这也会附加`Xcode`的调试器到`widget`的进程中。

<p align="center">
	<img style="width:281px;height:500px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-WidgetWithNothing-9-180x320.png" >
</p>
<br/>

看你的插件，碉堡了，对不对?虽然这是一个超级刺激的东西，但`widget`也需要一点工作。是时候让它做一些有趣的事了。

```
Note:你可能遇到在运行Widget的时候控制台打印出很多Auto Layout的错误。这是一个Xcode模板的问题，会有希望Apple在将来解决他。不要担心，大胆的添加AutoLayout约束到你的interface上。
（译者注：个人感觉还是纯代码靠谱，清晰，一目了然，版本冲突成本低）
```

### 建立接口

打开`MainInterface.storyboard`然后删除那个label。在`Size Inspector`中设置view的高为150pts，宽为320pts。从`Object Library`拖出一个`Button`，两个`Label`和一个`View`到你的`ViewController`的`view`上。

*   将其中一个label放在左上角，在`Attributes Inspector`中设置他的`Text`为`$592.12`，`Color`为`Red: 66, Green: 145，Blue: 211`。这个label会用来显示当前的市场价格。
*   将另外一个label放在你刚刚设置的label的右边，但在右边留出一个button的空间。在`Attributes Inspector`中设置他的`Text`为`+1.23`，`Color`为`Red: 133, Green: 191, Blue: 371`。这个会哦你过来显示昨天和当前的价格的不同。
*   将button移动到视图的右上方。在`Attributes Inspector`中设置他的`Image`为`caret-notification-center`，删除他的`Title`。
*   最后，将空的view放在两个label和button的下面，拉伸它直到它的底部和边缘和`containing view`吻合，设置他的`Height`为`98`.在`Attributes Inspector`中设置他的`Background`为`Clear Color`，然后在`Identity Inspector`中设置他的`Class`为`JBLineChartView`。

```
Note: 当你在打字的时候Xcode会猜测你可能输入的一个叫 JBLineChartDotView 的class，确保你选择了JBLineChartView
```
现在视图和`Outline`看起来像这样：

<p align="center">
	<img style="width:281px;height:500px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-DocumentOutline-10-480x183.png" >
</p>
<br/>

```
Note: 为了本书的清晰度的目的展示在这里的View是白色背景的。你的视图实际上会和通知中心一样有一个深色的背景色。
```
不要担心页面的布局，你马上就会通过适当的定义布局来添加AutoLayout约束。

现在展开在`Project Navigator`的`Crypticker`组，选中`Images.xcassets`。在`File Inspector`中勾选`BTC Widget`将其添加到extension的target上。
这样Xcode就会将`Images.xcassets`从你的`Crypticker`target上添加到`BTC Widget`的target上；这是你的button使用的`caret-notification-center`图片存放的地方。如果你在主app和widget上有重复的image assets，这是一个很好的共享方式。这可以通过不添加已经在使用的图片来减少App膨胀。


<p align="center">
	<img style="width:281px;height:500px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-AssetCatalog-11-480x273.png" >
</p>
<br/>

返回到`MainInterface.storyboard`，打开`Assistant Editor`。确保`TodayViewController.swift`是它的active file。将下面的代码添加到`TodayViewController.swift`的顶部：

```swift
import CryptoCurrencyKit
```
这是用来导入CryptoCurrencyKit framework。
接下来，你需要像这样更新他的类声明：
```swift
class TodayViewController: CurrencyDataViewController, NCWidgetProviding {
```
这会让`TodayViewController`成为`CurrencyDataViewController`的一个子类，确保他





---
also known as --> 又称,亦叫 

present information  --> 呈现

be a great way to  --> 是一个好方法

infancy --> 初期；婴儿期；幼年

Aside from --> 除…以外

peer-to-peer --> P2P,点对点

exchanges --> 交换

purchases --> 购买

except for --> 除了…以外

limitations --> 局限性

entire --> 整个

After all --> 毕竟; 究竟; 归根结底; （解释或说明理由）别忘了; 

Theoretically --> 理论上

Therefore --> 因此

appropriately --> 适当的

look something like this --> 看起来像这样

prompted --> 提示

activate --> 激活

named after --> 以...命名