---
layout: post
title: "<译>iOS 8 Today Extension Tutorial"
date: 2014-10-22 23:41:28 +0800
comments: true
categories: 
---
**原文链接:**[iOS 8 Today Extension Tutorial](http://www.raywenderlich.com/83809/ios-8-today-extension-tutorial)


**Ray提醒: ** 这篇文章可以说是一个精简版，它是从[iOS8 by Tutirials](http://www.raywenderlich.com/store/swift-tutorials-bundle)这本书中摘录的一个章节。[iOS8 by Tutirials](http://www.raywenderlich.com/store/swift-tutorials-bundle)是我们[iOS8 Feast](http://www.raywenderlich.com/82230/introducing-ios-8-feast)套餐中的一部分。通过这篇文章我们可以看到书中的内容是怎么样的。希望你喜欢！

iOS8介绍了一种新的概念 `App Extensions`:一种在操作系统上与其他应用共享应用程序功能的方式！

这些`Extensions`中有一种叫`Today Extensions`，又称`Widgets`。它可以让你在`Notification Center`中呈现信息。这是一种很好的方式向用户直接的提供他感兴趣的最新的信息。

在这个教程中，你会编写一个`Today Extension`用来渲染[bitcoin.org](https://bitcoin.org/en/)中比特币基于美元的当前市场价格。
<!-- more -->
##介绍比特币
如果你对比特币还不熟悉，那用一句很短的话来介绍的话就是比特币是一种还处于初期的数字加密货币。除了可以用来作P2P的交换和购买外，比特币交易还允许用户将比特币交易成另外一种数字加密货币，比如狗狗币和莱特币，以及像美金和欧元这样的常用货币。

##介绍Crypticker项目
一旦你开始写一个`Extension`，你必须先需要一个`载体应用`（host app）来做扩展；看看`Crypticker`项目吧。

`Crypticker`是用来展示当前的比特币价格、当前价格与昨日价格的区别和历史价格图表的一个简单的App。图表包括30天的历史价格；滑动你的手指可以看到过去特定的某天的准确价格。

`Extension`会包含所有这些功能，除了点击图表可以看到特定某天的价格。`Today Extension`也有局限性，尤其是当它在滑动(`swipe`)的时候，`Swipe Gesture`总是在`Notification Center`中的`Today`和`Notifications`这两个`sections`里被触发，所以他没有真的做到提供了最好的或者最可靠的用户体验。

## 入门指南

我们从下载[Crypticker起始项目](http://cdn4.raywenderlich.com/wp-content/uploads/2014/09/crypticker-swift-starter-GM.zip)入手。这个项目包含整个如上面描述的`Crypticker` app。我们的教程不会关注这个项目本身的开发，所以你愉快又惊讶的发现这个教程是如此的简洁。毕竟，你要写一个`Extension`，而不是整个app。
编译运行。请注意你需要一个可以工作的网络来链接`web service`以及拉取实时的价格。

(译者注：这个App还是Swift1.0的代码，可以根据Xcode的提示将代码修改成符合Swift1.1，主要是Swift1.1加入了可失败初始化功能)

<p align="center">
	<img style="width:444px;height:788px;" src="/images/custom/post/ios-8-today-extension-tutorial/CryptickerApp-1.png" >
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
	<img style="width:275px;height:488px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-Widget-2.png" >
</p>
<br/>

### 添加一个Today Extension target

所有的`Extention`被包装成一个和他们载体工程隔离的`binary`。所以你需要为你的`Crypticker`工程添加一个`Today Extention`的`target`。

在Xcode的`Project Navigator`中，选中`Crypticker project`，然后选择`Editor\Add Target…`来添加一个新的`target`。当模板选择器出现后，选择`iOS\ Application Extension`下的`Today Extension`。点击`Next`。

<p align="center">
	<img style="width:700px;height:411px;" src="/images/custom/post/ios-8-today-extension-tutorial/AddTarget-3-700x411.png" >
</p>
<br/>

设置`Product Name`为`BTC Widget`，然后验证`Language`为`Swift`，`Project`为`Crypticker`，`Embed in Application`也是`Crypticker`。点击`Finish`。


<p align="center">
	<img style="width:700px;height:413px;" src="/images/custom/post/ios-8-today-extension-tutorial/AddTarget-4-700x413.png" >
</p>
<br/>

当提示激活`BTC Widget``scheme`的时候.就像文本指示器一样，会为你创建另一个`Xcode``scheme`。

恭喜！现在`BTC Widget`会展示在你的`targets`的列表中。


<p align="center">
	<img style="width:250px;height:250px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-AddTarget-5-250x250.png" >
</p>
<br/>

确保你选中了`BTC Widget`target，选择`General`标签，然后按下在`Linked Frameworks and Libraries`下的`+`按钮。

<p align="center">
	<img style="width:480px;height:294px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-LinkFramework-6-480x294.png" >
</p>
<br/>

选中`CryptoCurrencyKit.framework`然后点击`Add`。

CryptoCurrencyKit是一个在`Crypticker app`中使用的用来从网络上获取货币价格的自定义的`framework`。你很幸运，有个难以置信般善良和体贴的程序猿模块化了`Crypticker`的代码到`framework`，所以它可以在多个`target`下共享 :]

为了在载体工程和它的extensions质检共享代码，你必须使用自定义`framework`。如果你不这么做，你会发现你重复撰写了很多代码以及违反了软件工程中一个很重要的原则：`DRY- or，Dont Repeat Yourself`，我再说一次：`Dont Repeat Yourself`（不要重复发明轮子）

现在，你已经准备好开始实现`extension`了。

注意，现在有一个以你新的`target`命名的文件组在你项目的`Navigator`中，`BTC Widget`。这是`Today Extension`代码默认分组存放的地方。

展开这个文件组，你会看到这里有一个`view controller`，一个`storyboard`文件和一个`Info.plist`文件。它的`target`配置信息也告诉他去`MainInterface.storyboard`加载他的`interface`，`MainInterface.storyboard`包含一个指定class为 `TodayViewController.swift`的 `ViewController`。

<p align="center">
	<img style="width:274px;height:320px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-ListofFiles-7-274x320.png" >
</p>
<br/>

你会发现有一些你期望看到的文件在`Today Extension`模板中却消失不见了。比如说`app delegate`。记住所有的Extension都运行在另外一个载体工程的内部，所以他们和传统的app的生命周期不一样。

本质上，Extension的生命周期是被映射在`TodayViewController`的生命周期上的。

打开`MainInterface.storyboard`。你会看到一个带着明亮的`Hello World``label`的深色的`View`。为了与通知中心的暗色调协调，当`Today Extension`有一个清晰透明的背景和一个明亮或者艳丽的文本时是最清晰易读的。

确保`BTC Widget`scheme在`Xcode`的`toolbar`中被选中，然后编译运行。这时会出现一个窗口来问你哪一个app需要运行。Xcode正在问你哪个载体应用需要运行。选择`Today`。这会告诉你的iOS打开通知中心的`Today`视图。它会依次展开你的`Widget`。

<p align="center">
	<img style="width:349px;height:320px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-PickAppToRun-8-349x320.png" >
</p>
<br/>

这也会附加`Xcode`的调试器到`widget`的进程中。

<p align="center">
	<img style="width:180px;height:320px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-WidgetWithNothing-9-180x320.png" >
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
	<img style="width:480px;height:183px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-DocumentOutline-10-480x183.png" >
</p>
<br/>

```
Note: 为了本书的清晰度的目的展示在这里的View是白色背景的。你的视图实际上会和通知中心一样有一个深色的背景色。
```
不要担心页面的布局，你马上就会通过适当的定义布局来添加AutoLayout约束。

现在展开在`Project Navigator`的`Crypticker`组，选中`Images.xcassets`。在`File Inspector`中勾选`BTC Widget`将其添加到extension的target上。
这样Xcode就会将`Images.xcassets`从你的`Crypticker`target上添加到`BTC Widget`的target上；这是你的button使用的`caret-notification-center`图片存放的地方。如果你在载体app和widget上有重复的image assets，这是一个很好的共享方式。这可以通过不添加已经在使用的图片来减少App膨胀。


<p align="center">
	<img style="width:480px;height:273px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-AssetCatalog-11-480x273.png" >
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
这会让`TodayViewController`成为`CurrencyDataViewController`的一个子类，确保他符合`NCWidgetProviding`协议。

`CurrencyDataViewController`是包含在`CryptoCurrencyKit`中也被`Crypticker`载体应用的第一个视图使用的Controller。由于Widget和App会通过UIViewController展示相似的信息，他会把可重用的组件放到一个superclass中，然后按不同的需求来做子类化。

`NCWidgetProviding`是`Widget`特有的接口。有两个接口的方法需要你来实现。

按住`Ctrl`，从IB的Button拖动鼠标到类声明下。在弹出的对话框中确保`Connection`设置为`Outlet`，`Type`设置为`UIButton`，在`Name`中输入`toggleLineChartButton`后点击`Connect`。

<p align="center">
	<img style="width:480px;height:192px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-WireToggleButton-480x192.png" >
</p>
<br/>

然后还是按住Ctrl，这次从IB的Button拖动鼠标到类的底部。在弹出的对话框中改变`Connection`值为`Action`，设置`Type`为`UIButton`，在`Name`中输入`toggleLineChart`后点击`Connect`。

<p align="center">
	<img style="width:480px;height:196px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-WireUpAction-480x196.png" >
</p>
<br/>

`TodayViewController`是`CurrencyDataViewController`的子类，`CurrencyDataViewController`有三个`outlet`：`price label`, `price change label` 和 `line chart view`。你现在需要把他们联通起来。在`Document Outline`中按住`Ctrl`，从`Today View Controller`拖拽到`price label`(设置他的text为$592.12).在弹出框中选中`priceLabel`来建立连接。对另外一个label也做重复的操作，选中弹出框中的`priceChangeLabel`.最后对 `Line Chart View`做同样的操作，在弹出框中选中`lineChartView`。


<p align="center">
	<img style="width:480px;height:222px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-WiringMiscThingsUp-480x222.png" >
</p>
<br/>

### Auto Layout

为了你的Widget可以自适应，你就会需要对它设置`Auto Layout`的约束。iOS8有一个新的自适应布局的概念。大意是视图可以被设计成各自有一个单独的布局，并且这个布局可以在各种各样的屏幕大小上工作。视图被设计成可以在未来以及不知道材料的设备上都可以自适应。

其中一个你要添加的约束是用来显示和隐藏我们的图表以及用来帮助定义Widget整体的高度。通知中心会依赖于你定义的适当的高度来显示你的Widget。

选中$592.12这个label，然后选择`Editor\Size to Fit Content`。如果`Size to Fit Content`选项不在你的菜单中，取消选中这个label，然后再次选中重试一遍。Xcode有的时候会抽风。接下来，使用在storyboard画布下的`Pin`按钮。分别钉住`Top`和`Leading`的空间为8和16。确保`Constrain to margins`处于关闭状态。

<p align="center">
	<img style="width:208px;height:320px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-AutoLayout-1-208x320.png" >
</p>
<br/>

现在选中+1.23这个label，再次选中`Editor\Size to Fit Content`。然后使用`Pin`按钮，钉住`Top`和`Trailing`空间都为8.

<p align="center">
	<img style="width:209px;height:320px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-AutoLayout-2-209x320.png" >
</p>
<br/>

选中那个Button，使用`Pin`按钮，钉住他的`Top`和`Trailing`两个的空间为0，以及他的Button空间为8。再钉住他的`Width`和`Height`都为44.确保`Constrain to margins`处于关闭状态。

<p align="center">
	<img style="width:206px;height:320px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-AutoLayout-3-206x320.png" >
</p>
<br/>

你需要减小Button的底部空间约束的优先级。选中button，然后打开`Size Inspector`, 定位到`Bottom Space to`：约束列表的约束，点击`Edit`然后改变他的`Priority`为250.

通过降低优先级你被`Auto Layout`系统允许打破他认为必要的约束。250是一个被设置给所有的约束优先级的默认以及必须的且碰巧小于1000的任意值。在折叠状态下这个约束需要被打破。通过设置不同优先级的约束你暗示系统当发生冲突的时候哪一个约束最先或者最后打破。


<p align="center">
	<img style="width:480px;height:303px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-AutoLayout-4-480x303.png" >
</p>
<br/>

最后，选中`Line Chart View`。使用`Pin`按钮，钉住他的`Leading`, `Trailing`和`Bottom`三个空间为0以及他的高度为98。


<p align="center">
	<img style="width:209px;height:320px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-AutoLayout-5-209x320.png" >
</p>

从`Document Outline`中选中视图控制器的View，然后选择`Editor\Resolve Auto Layout Issues\All Views\Update Frames`.这会通过更新视图的frame来匹配他们的约束，从而修复在画布上的任何`Auto Layout`的警告。如果`Update Frames`不可用，那么你的一切都狠完美，这是不必要的运行。

完成了你的所有的约束后，最后一步就是为`Line Chart View`的高度创建一个outlet的约束。在`Document Outline`中找到`Line Chart View`然后点击显示的三角形。

然后点击三角形，找到我们需要的高度约束。选中他，然后按住Ctrl，拖拽到 `Assistant Editor`，放在其他outlet的下面。在弹出框中确保`Connection`被设置为`Outlet`，在`Name`中输入`lineChartHeightConstraint`。点击`Connect`。

<p align="center">
	<img style="width:480px;height:190px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-AutoLayout-6-480x190.png" >
</p>

### 实现TodayViewController.swift

现在接口已经确立了，而且所有的东西都已经连接上，在`Standard Editor`中打开 `TodayViewController.swift`。
你会发现你在与一个普通的UIViewController的子类打交道。很欣慰，对不对？一会儿后你会遇到`NCWidgetProviding`协议中的一个叫做`widgetPerformUpdateWithCompletionHandler`的方法。在这个教程的结尾你会学到更多与它相关的东西。

这个`ViewController`主要负责显示当前的价格，价格的区别，响应button的点击和在line chart中显示价格的历史。

在`TodayViewController`的顶部定义一个属性用来跟踪线形图（line chart）是否可见：

```swift
var lineChartIsVisible = false
```
现在用下面的实现替换掉`viewDidLoad()`的模板代码：

```swift
override func viewDidLoad() {
  super.viewDidLoad()
  lineChartHeightConstraint.constant = 0
 
  lineChartView.delegate = self;
  lineChartView.dataSource = self;
 
  priceLabel.text = "--"
  priceChangeLabel.text = "--"
}
```

这个方法会做以下几件事情：

	1.设置线形图的height约束的约束值为0，所以线形图默认是hidden的。
	2.设置线形图的dataSource和delegate为self。
	3.设置一些占位文案在两个label上。

还是在`TodayViewController`中，添加如下方法：

```swift
override func viewDidAppear(animated: Bool) {
  super.viewDidAppear(animated)
 
  fetchPrices { error in
    if error == nil {
      self.updatePriceLabel()
      self.updatePriceChangeLabel()
      self.updatePriceHistoryLineChart()
    }
  }
}
```

其中的`fetchPrices`方法是在`CurrencyDataViewController`中定义的，他是一个可以异步调用block。这个方法会向我们在这个教程开始的时候提到的`web-service`发送一个请求来获取比特币的价格信息。

在方法的实现块中更新所有的label和线形图。更新方法也在父类中为你定义好了。他们通过`fetchPrices`对需要的值做一个简单的检索以及合理的格式化后用来显示。

由于Widget的设计，你还需要实现`widgetMarginInsetsForProposedMarginInsets`方法用来提供自定义的 `margin insets`，添加如下代码到`TodayViewController`中：

```swift
func widgetMarginInsetsForProposedMarginInsets
  (defaultMarginInsets: UIEdgeInsets) -> (UIEdgeInsets) {
    return UIEdgeInsetsZero
}
```
Widget默认有一个很大的`left margin`，这在Apple很多的默认Widget上是恨明显的。如果你想要填充整个通知中心的宽度，你必须实现这个方法并且返回`UIEdgeInsetsZero`，他会把所有的边的`margin insets`都转化为0。

现在是时候看看你到目前为止都有些啥了。选中`BTC Widget`这个`Scheme`。编译运行。当app运行后提示的时候选择`Today`。

	*如果通知中心没有显示，那么在屏幕的顶部用手指向下滑动来激活它。
	*如果Widget没有在通知中心显示，你需要通过编辑菜单来添加它。在今日视图的底部你会看到编辑按钮。点击按钮展开所有安装在系统中的Widget。在这里你可以随心所欲的enable, disable以及re-order他们。如果BTC Widget没有处于enable状态，则enable他。


<p align="center">
	<img style="width:480px;height:164px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-Widget-Almost1-480x164.png" >
</p>


屌爆了，有木有！你的Widget现在已经在通知中心实时的显示比特币的价格。但是你可能已经意识到了一个问题。按钮不能工作以及看不到线形图。


<p align="center">
	<img style="width:480px;height:329px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-YUNO.png" >
</p>

接下来，你要为你添加的button实现`toggleLineChart`方法，用来展开widget的视图并且显示线形图。就像方法名说的一样，这个button就像一个转换键一样；他也会折叠视图从而隐藏图表。

用下面的代码替换空的`toggleLineChart`：

```swift
@IBAction func toggleLineChart(sender: UIButton) {
  if lineChartIsVisible {
    lineChartHeightConstraint.constant = 0
    let transform = CGAffineTransformMakeRotation(0)
    toggleLineChartButton.transform = transform
    lineChartIsVisible = false
  } else {
    lineChartHeightConstraint.constant = 98
    let transform = CGAffineTransformMakeRotation(CGFloat(180.0 * M_PI/180.0))
    toggleLineChartButton.transform = transform
    lineChartIsVisible = true
  }
}
```
这个方法用来操纵线形图的高度约束，从来约束他的显示。它也对button有一个旋转变化，所有它可以准确的反应出图表的可见度。

约束更新后，你必须重新加载图表的数据，以便它在新的约束上重新绘制。

你会在`viewDidLayoutSubviews`上做这些。添加如下代码到`TodayViewController`：

```swift
override func viewDidLayoutSubviews() {
  super.viewDidLayoutSubviews()
  updatePriceHistoryLineChart()
}
```

确保BTC Widget这个scheme被选中。编译运行。当App运行的时候出现提示就选择`Today`。

在左图，你会看到当图表隐藏的时候widget是如何展示的。在右图，你会看到当图表打开时widget是如何展示的。不是很寒酸！


<p align="center">
	<img style="width:480px;height:144px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-Widet-Almost2-480x144.png" >
</p>

添加如下代码到`TodayViewController`，你会拥有一个快速更新的线颜色以及目光尖锐的widget：

```swift
override func lineChartView(lineChartView: JBLineChartView!,
  colorForLineAtLineIndex lineIndex: UInt) -> UIColor! {
    return UIColor(red: 0.17, green: 0.49,
      blue: 0.82, alpha: 1.0)
}
```

确保当前的scheme依旧选中，编译运行。当app运行的时候出现提示就选择`Today`。

<p align="center">
	<img style="width:480px;height:286px;" src="/images/custom/post/ios-8-today-extension-tutorial/BTC-Widet-Done-480x286.png" >
</p>

你最后要做的就是通过允许系统来创建一个快照，当屏幕关闭的时候添加对Widget更新视图的支持。系统会定期的帮助你的widget保持最新。

用下面的代码替换掉`widgetPerformUpdateWithCompletionHandler`现存的实现：

```swift
func widgetPerformUpdateWithCompletionHandler(completionHandler: ((NCUpdateResult) -> Void)!) {
  fetchPrices { error in
    if error == nil {
      self.updatePriceLabel()
      self.updatePriceChangeLabel()
      self.updatePriceHistoryLineChart()
      completionHandler(.NewData)
    } else {
      completionHandler(.NoData)
    }
  }
}
```

这个方法做以下几件事情：
	
	*他通过调用fetchPrices来从网络服务中获取当前的价格数据。
	*如果没有错误，接口就会更新数据。
	*最后 - 就像NCWidgetProviding协议要求的一样 - 方法调用系统提供的带着.NewData枚举的block。
	*发生错误的时候，block会带着.Failed枚举调用。这会告诉系统没有新的可用的数据，用现存的快照吧，亲！
	
你可以在[这里](http://cdn3.raywenderlich.com/wp-content/uploads/2014/09/crypticker-swift-final-GM.zip)下载最后的工程.

### 现在去干哈

iOS8通知中心是你自己个人的`playground`！Widget在一些其他的手机操作系统上早就已经有了，Apple最终提供给你一个创造他们的能力。

作为一个有进取心的开发者，你可能想要再看看你现在的app，思考怎么用widget去提升他们。利用widget的可能性更进一步想象新的app的idea。

如果你喜欢学习更多关于创建其他类型的iOS8的App扩展。看看我们的书 [iOS 8 by Tutorials](http://www.raywenderlich.com/?page_id=74805)，这里你可以学习关于`Photo Extensions`, `Share Extensions`, `Action Extensions`以及更多的知识。


我们迫不及待的想看你能想出什么东西来了，希望你的Widget尽快的在我们的通知中心的顶部！
