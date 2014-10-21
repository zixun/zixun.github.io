---
layout: post
title: "iOS学习资料整理"
date: 2014-03-06 19:24:23 +0800
comments: true
categories: 
---

**NOTE:** 这篇文章主要用来收集自己在网上看到的不错的文章，并做归类整理，这篇文章会持续更新，确保每篇文章都是自己看过的，也可以当做是自己的学习资料。<br/><br/>
**目录** <br/>

*   [Swift](#swift)
*   [屏幕适配](#screen_adaptation)
*   [工程](#application)
*   [Foundation](#foundation)
*   [旋转](#rotation)
*   [Category](#category)
*   [UIKit](#uikit)
	*   [UIViewController](#uiviewcontroller)
	*   [UITableView](#uitableview)
	*   [UIActionSheet](#uiactionSheet)
	*   [其他](#uikitother)
*   [iOS7](#ios7)
*   [内存管理](#memory)
*   [Block](#block)
*   [GCD](#gcd)
*   [绘图&动画](#graphics&animation)
*   [runtime](#runtime)
*   [c语言](#c-language)
*   [代码优化](#uplevel)
*   [调试](#debug)
*   [第三方库使用](#vender)
*   [底层分析](#analyze)
*   [StoryBoard](#storyBoard)
*   [面试](#interview)
*   [工具](#tools)
*   [技巧](#xiaojiqiao)
*   [博客](#blog)
*   [期刊](#periodical)
*   [Markdown](#markdown)

<br/>
<!-- more -->

<h1 id="swift">Swift</h1>

* [The Swift Programming Language 中文版](http://numbbbbb.gitbooks.io/-the-swift-programming-language-/content/) by Swift 中文翻译组<br/>

* [CocoaChina Swift专场](http://www.cocoachina.com/special/swift/) by CocoaChina<br/>

* [Swift语言指南](https://github.com/ipader/SwiftGuide) by ipader<br/>

* [喵神的Swift书~Tip列表](http://swifter.tips/) by onevcat<br/>

* [Swift论坛社区](https://swiftist.org/) by Swiftist<br/>



<h1 id="screen_adaptation">屏幕适配</h1>

* [为iPhone6设计自适应布局（一）](http://www.devtalking.com/articles/adaptive-layout-for-iphone6-1/) by 宇轩<br/>

* [为iPhone6设计自适应布局（二）](http://www.devtalking.com/articles/adaptive-layout-for-iphone6-2/) by 宇轩<br/>


<h1 id="application">工程</h1>
* [iOS开发实战：如何将非ARC的项目转换成ARC项目](http://www.cocoachina.com/ios/20140912/9605.html) by CocoaChina<br/>


<h1 id="foundation">Foundation</h1>
* [对NSArray中自定义的对象进行排序](http://beyondvincent.com/blog/2014/01/26/how-to-sort-nsarray-with-custom-objects/) by破船<br/>

* [Compile-time Key Paths Verification](http://lldong.github.io/2014/02/24/key-paths-validation.html) by lldong<br/>

* [Cocoa 漢字轉拼音最簡單的方法](http://lldong.github.io/2012/11/06/hanzi-to-pinyin.html) by lldong<br/>

* [iOS将大文件映射到内存](http://blog.xcodev.com/archives/ios-mmap-api/) by xcodev<br/>

* [用宏提速NSCoding](http://www.isaced.com/post-244.html) by isaced<br/>
* [Toll-Free Bridging](http://gracelancy.com/blog/2014/04/21/toll-free-bridging/) by Lancy<br/>

<h1 id="rotation">旋转</h1>
* [iOS旋转视图实践](http://rdc.taobao.org/?p=408) by淘宝技术部<br/>
* [强制旋转一个UIViewController](http://blog.t-xx.me/blog/2014/01/19/force-rotate-uiviewcontroller/) bytxx's blog<br/>

<h1 id="category">Category</h1>
* [Objective-C相关Category的收集](http://www.cocoachina.com/applenews/devnews/2014/0212/7808.html) by CocoaChina<br/>

* [Objective-C语言在Category中实现属性](http://blog.xcodev.com/archives/implement-objc-property-in-category/) by xcodev<br/>

<h1 id="uikit">UIKit</h1>

<h4 id="uiviewcontroller">UIViewController</h4>
* [iOS5中UIViewController的新方法](http://blog.devtang.com/blog/2012/02/06/new-methods-in-uiviewcontroller-of-ios5/) by 唐巧<br/>


<h4 id="uitableview">UITableView</h4>
* [UITableView 滚动流畅性优化](http://blog.cocoabit.com/blog/2014/02/09/uitableview-gun-dong-liu-cheng-xing-you-hua/) by 6david9<br/>

* [给tableview Cell添加动画](http://beyondvincent.com/blog/2014/01/13/animation-tableview-cell/) by 破船<br/>

* [利用长按手势移动 Table View Cells](http://beyondvincent.com/blog/2014/03/26/cookbook-moving-table-view-cells-with-a-long-press-gesture/) by 破船<br/>

* [制作一个可以滑动操作的 Table View Cell](https://github.com/nixzhu/dev-blog/blob/master/2014-04-26-make-swipeable-table-view-cell-actions-without-going-nuts-scroll-views.md) by nixzhu<br/>



* [TableView中嵌套一个ScrollView有时导致ScrollView无法滚动](http://blog.xcodev.com/archives/cant-scroll-in-tableview/) by xcodev<br/>

<h4 id="uiwebview">UIWebView</h4>
* [iOS5网页视图（UIWebView）中的输入框不能弹出键盘的问题](http://blog.xcodev.com/archives/no-keyboard-in-webview-of-ios5/) by xcodev<br/>

<h4 id="uiactionSheet">UIActionSheet</h4>
* [封装同步的UIActionSheet](http://blog.devtang.com/blog/2012/06/24/enhance-uiactionsheet/) by 唐巧<br/>

<h4 id="uikitother">其他</h4>
* [如何自定义iOS中的控件](http://beyondvincent.com/blog/2014/01/20/how-to-build-a-custom-control-in-ios/) by 破船<br/>


<h1 id="ios7">iOS7</h1>
* [iOS 7中实现模糊效果](http://beyondvincent.com/blog/2014/01/29/ios-7-blur-effects-gpuimage/) by破船<br/>

* [iOS 7 教程：让程序同时支持iOS 6和iOS 7](http://beyondvincent.com/blog/2013/11/19/122-working-with-ios-6-and-7/) by破船<br/>

* [iOS 7 教程：浅析Text Kit](http://beyondvincent.com/blog/2013/11/12/121-brief-analysis-text-kit/) by破船<br/>

* [iOS 7 教程：定制iOS 7中的导航栏和状态栏](http://beyondvincent.com/blog/2013/11/03/120-customize-navigation-status-bar-ios-7/) by破船<br/>

* [iOS 7中的一些小修改](http://beyondvincent.com/blog/2013/09/20/112-ios-7-additions-omg-finally/) by破船<br/>

* [iOS 7 键盘动画](http://nonomori.farbox.com/post/ios-7-jian-pan-dong-hua) by nonomori<br/>

<h1 id="memory">内存管理</h1>
* [手把手教你ARC——iOS/Mac开发ARC入门和使用](http://onevcat.com/2012/06/arc-hand-by-hand/) by onevcat<br/>

* [retainCount 不会为 0](http://lldong.github.io/2011/10/20/retain-count.html) by lldong<br/>


 
<h1 id="block">Block</h1>
* [谈Objective-C Block的实现](http://blog.devtang.com/blog/2013/07/28/a-look-inside-blocks/) by 唐巧<br/>

<h1 id="gcd">GCD</h1>
* [使用GCD](http://blog.devtang.com/blog/2012/02/22/use-gcd/) by 唐巧<br/>
* [GCD 深入理解：第一部分](https://github.com/nixzhu/dev-blog/blob/master/2014-04-19-grand-central-dispatch-in-depth-part-1.md) by nixzhu<br/>

<h1 id="graphics&animation">绘图&动画</h1>
* [在iOS中让图片旋转时抗锯齿](http://blog.xcodev.com/archives/anti-alise-for-image-ios/) by xcodev<br


* [[Objective C中C99的使用](http://answerhuang.duapp.com/index.php/2013/10/17/objective-c_c99/) by answerhuangbr/>
*
<h1 id="uplevel">代码优化</h1>
* [类簇在iOS开发中的应用](http://blog.leezhong.com/ios/2014/01/04/class-cluster.html) by无网不剩<br/>

* [iOS项目的目录结构和开发流程](http://blog.leezhong.com/ios/2013/09/23/build-ios-application.html) by无网不剩<br/>



<h1 id="runtime">runtime</h1>
* [objc/runtime 探索(一)](http://blog.devwu.com/develop/2014-08-15/objcruntime-explore1/)<br/>
* [objc/runtime 探索(二))](http://blog.devwu.com/develop/2014-08-17/objcruntime-explore2/)<br/>
* [objc/runtime 探索(三))](http://blog.devwu.com/develop/2014-08-18/objcruntime-explore3/)<br/>
* [objc/runtime 探索(四))](http://blog.devwu.com/develop/2014-08-19/objcruntime-explore4/)<br/>

<h1 id="c-language">c语言</h1>
* [什么情况下用宏定义do{}while(0);这种结构体](http://www.cnblogs.com/rollenholt/articles/1907414.html)<br/>

<h1 id="debug">调试</h1>
* [LLDB调试命令初探](http://www.starfelix.com/blog/2014/03/17/lldbdiao-shi-ming-ling-chu-tan/) by 达叔<br/>

<h1 id="vender">第三方库使用</h1>
* [在iOS开发中使用FMDB](http://blog.devtang.com/blog/2013/07/28/a-look-inside-blocks/) by 唐巧<br/>


<h1 id="analyze">底层分析</h1>
* [Objective-C对象模型及应用](http://blog.devtang.com/blog/2013/10/15/objective-c-object-model/) by 唐巧<br/>


<h1 id="storyBoard">StoryBoard</h1>
* [StoryBoard--看上去很美](http://blog.devtang.com/blog/2012/12/15/do-not-use-storyboard/) by 唐巧<br/>


<h1 id="interview">面试</h1>
* [上级向的十个iOS面试问题](http://onevcat.com/2013/04/ios-interview/) by onevcat<br/>


<h1 id="tools">工具</h1>
* [使用CocoaPods来做iOS程序的包依赖管理](http://blog.devtang.com/blog/2012/12/02/use-cocoapod-to-manage-ios-lib-dependency/) by 唐巧<br/>

* [CocoaPods详解之----制作篇](http://blog.csdn.net/wzzvictory/article/details/20067595) by wangzz<br/>


* [Enrolling in Apple Developer Programs](http://girlios.github.io/blog/2014/03/16/enrolling-in-apple-developer-programs/) by Girl_iOS<br/>

<h1 id="xiaojiqiao">技巧</h1>
* [Xcode Key Bindings & Gestures利用快捷键提高开发效率](http://www.cocoachina.com/newbie/basic/2014/0225/7882.html) by CocoaChina<br/>

* [www.gitignore.io/](http://www.gitignore.io/) by gitignore<br/>
  这个网站可以自动为你生成gitignore文件，比如你输入Objective Xcode 就会为你生成正对iOS开发的gitignore文件，如果你对gitignore文件不熟，那就请翻阅<Git权威指南>。

* [Chrome 快捷键 整理版 【来自豆瓣】](http://www.douban.com/note/165306479/) by douban<br/>

* [少有人知的 GitHub 使用技巧](http://segmentfault.com/a/1190000000475547) by segmentfault<br/>

<h1 id="pulgin">插件</h1>

* [Alcatraz](https://github.com/supermarin/Alcatraz) by Github</br>
Alcatraz是一个帮你管理Xcode插件、模版以及颜色配置的工具。它可以直接集成到Xcode的图形界面中，让你感觉就像在使用Xcode自带的功能一样.这里有一篇很好的介绍它的文章[使用Alcatraz来管理Xcode插件](http://blog.devtang.com/blog/2014/03/05/use-alcatraz-to-manage-xcode-plugins/)
</br>



* [ColorSense-for-Xcode](https://github.com/omz/ColorSense-for-Xcode) by Github<br/>
这个插件可以在编辑器上动态的渲染出你代码编写的颜色，例如在你写如下代码时它会在这段代码的右上角绘制出颜色预览，可以省去很多UI调整的时间</br>
<img style="position:absolute; left:50%; margin-left:-375px;" src="/images/custom/post/ColorSense.png" >
<br/>

* [KSImageNamed-Xcode](https://github.com/ksuther/KSImageNamed-Xcode) by Github<br/>
这个工具可以帮你自动补全image的图片信息，效果非常惊艳, 预览，自动补全，提示有无@2x高清图。</br>
<p><img style="position:absolute; left:50%; margin-left:-271px;" src="/images/custom/post/test.png" ></p>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>




<h3 id="blog">博客</h3>

* [Girl_iOS](http://girlios.github.io/)(萌妹子，iOS程序媛，下面的大牛博客很多都是摘自她的博客@Girl_iOS)
* [唐巧](http://blog.devtang.com/)(InfoQ编辑，Blogger，iOS开发，创业者，前网易员工。微信公共账号iOSDevTips创建者。)
* [破船](http://beyondvincent.com/)(宠辱不惊，闲看庭前花开花落。去留无意，漫随天外云卷云舒。不妄取，不妄予，不妄想，不妄求。与人方便，随遇而安)
* [喵神](http://onevcat.com/) (iOS/Mac,Unity3D开发者，现就职于日本创意公司Kayac，正在修行，探求创意之源)
* [念茜](http://nianxi.net/)(一单线程妞儿，iOS安全大牛)
* [宇轩](http://www.devtalking.com/)(付宇轩，80后，程序员，关注技术和人文，记录学习的点点滴滴于《程序员说》)
* [6david9](http://blog.cocoabit.com/)(iOS码农，攻城狮。喜欢各种有意思的东西。最近迷恋上了自行车。)
* [txx’s blog](http://blog.t-xx.me/)(中山大学大四翘课熬夜党 广州贴贴信息科技技术总监 高度强迫症 代码洁癖的 iOS开发者)
* [lldong](http://lldong.github.io/)(不详！)
* [yingkong1987](http://yingkong1987.github.io/)(@兔be南玻1)
* [xcodev](http://blog.xcodev.com/)(资深iOS开发工程师@谌启亮)
* [isaced](http://www.isaced.com/)(iOS Programmer@isaced)
* [answerhuang](http://answerhuang.duapp.com/)(iOS developer, Python fans@answer-huang)
* [卢克](http://geeklu.com/)(Mac，iOS开发@卢小克)
* [余书懿](http://blog.csdn.net/ysy441088327)(代表作:<豆豆音乐> @余书懿)
* [Creator of moke](http://wangling.me/)(Creator of 墨客(moke.com) and Voodo(moke.com/voodo)@an00na)
* [萧宸宇](http://iiiyu.com/)(注定漂泊的人@Sumi-iYu)
* [webfrogs](http://webfrogs.me/)(iOS开发，开源爱好者 @webfrogs)
* [esoftmobile](http://esoftmobile.com/)(iOS开发者 esoftmobile.com @TracyYih)
* [无网不剩](http://blog.leezhong.com/)(iOS开发@李忠)
* [starfelix达叔](http://www.starfelix.com/)(不要告诉任何人你无法实现自己的梦想，包括我！@达叔是一种沧桑)
* [老谭](http://www.tanhao.me/)(不详，但看博客内容很牛！)








<h3 id="periodical">期刊</h3>
* [objc.io](http://www.objc.io/) ---中文版---> [objccn.io](http://www.objccn.io/)

* [nshipster.com](http://nshipster.com/) ---中文版---> [nshipster.cn](http://nshipster.cn/)


* [raywenderlich.com](http://www.raywenderlich.com/) 




<h3 id="markdown">Markdown</h3>
* [Markdown 语法说明 (简体中文版)](http://wowubuntu.com/markdown/index.html)<br/>

