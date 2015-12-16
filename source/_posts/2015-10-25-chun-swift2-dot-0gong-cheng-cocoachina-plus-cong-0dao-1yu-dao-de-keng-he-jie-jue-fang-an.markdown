---
layout: post
title: "纯Swift2.0工程CocoaChina+从0到1遇到的坑和解决方案"
date: 2015-10-25 00:03:58 +0800
comments: true
categories: 
---

好久没写技术文章了，感觉这个事情不应该被搁置，程序员应该努力去学一些除了写代码以外的东西。

前段时间博主从阿里巴巴跳槽加盟滴滴快的，乘着跳槽的时间差做了两件一直想做的事：
一件就是用Swift2.0写了一个CocoaChina的iOS app（名字叫CocoaChina+，文章的最后会介绍），另外一件就是带着女票去北京玩了一趟，见了见以前读硕时在实验室的几个哥们。整个app从无到有遇到了好多Swift的坑，只可惜没有一一记载下来，现在凭着记忆将还记得的坑以及好的东西记载下来与大家分享。

<!-- more -->

## 坑: dyld: Library not loaded: @rpath/libswiftCore.dylib

Swift代码在模拟器上跑的好好的，突然到真机就不行了，一起来就报错`dyld: Library not loaded: @rpath/libswiftCore.dylib`

#### 解决方案
点开你工程target的`Build Settings`，搜索`Embedded Content Contains Swift Code`，这个值默认是NO，设置为YES即可。
####参考
[http://stackoverflow.com/questions/26024100/dyld-library-not-loaded-rpath-libswiftcore-dylib/26949219#26949219](http://stackoverflow.com/questions/26024100/dyld-library-not-loaded-rpath-libswiftcore-dylib/26949219#26949219)

## 坑: Cannot assign to property:"xxx"is a get-only property
只读属性当然是不能设置的啦，博主是不是傻。这里说的情况有点不一样。比如我们有一份Obectice-C的代码，其中`Class A`有个只读属性`Class B`，`Class B`又有一个读写属性`Property P`，然后我们把这份OC代码放到我们的Swift工程中，用Swift去调用`A.B.P`,就会报错`Cannot assign to property:"xxx"is a get-only property`，有点类似Swift可选链的感觉，一但OC的类其中一个是只读的，接下去的就都是只读的了。为什么这里一直强调OC代码，因为博主试过Swift的代码就不存在这个问题。

#### 解决方案
写一个`SwiftFucker`OC类，在这个类里面去调用这行代码返回，恩，`SwiftFucker`！

	@interface SwiftFucker : NSObject

	+ (void)fuckSetIsAutoLoginEnabled;

	@end
	
	@implementation SwiftFucker

	+ (void)fuckSetIsAutoLoginEnabled {
    	//Swift中chatManager是readonly，会让他的属性IsAutoLoginEnabled也变成readonly
    	[[EaseMob sharedInstance].chatManager setIsAutoLoginEnabled:YES];
	}

	@end

Swift中就直接这样调：

	SwiftFucker.fuckSetIsAutoLoginEnabled()
    //EaseMob.sharedInstance().chatManager.isAutoLoginEnabled = true;
 

## 坑: 收到Apple的iTunes Store邮件说Invalid Swift Support
当我们用Xcode构建打包后提交到AppStore，然后准备喝杯咖啡，喝完看看iTunesConnect后台对我们的App处理完毕没，完毕了就可以提交审核了，可是千等万等一直提示你的构建`正在处理`，过了好一会你的开发者邮箱就收到了Apple的邮件说你的App有问题啊，不支持Swift啊：

	Invalid Swift Support - The files libswiftCoreAudio.dylib, libswiftCoreMedia.dylib don’t match /Payload/CocoaChinaPlus.app/Frameworks/libswiftCoreAudio.dylib, /Payload/CocoaChinaPlus.app/Frameworks/libswiftCoreMedia.dylib.
	Make sure the files are correct, rebuild your app, and resubmit it. Don’t apply post-processing to /Payload/CocoaChinaPlus.app/Frameworks/libswiftCoreAudio.dylib, /Payload/CocoaChinaPlus.app/Frameworks/libswiftCoreMedia.dylib.
	
####解决方案

其实Swift工程有个坑，就是Swift为了支持之前的OS版本，会将Swift相关的lib全部打包到我们的工程中，也就是上面列出来的libswift相关的库，所以你不妨试一下，新建一个OC的工程和一个Swift的工程，然后各自打包，OC的才几百K，Swift的5M，哎，啥都没干呢，就感觉被Swift干了~。这5M就是Swift运行时相关的lib。那上面Apple告诉我们app里找不到这几个库，其实是Cocoapods的bug，将我们的Cocoapods更新到目前最新的0.39.0即可。

####参考
[https://github.com/CocoaPods/CocoaPods/issues/4188](https://github.com/CocoaPods/CocoaPods/issues/4188)

#####------------------我是可爱的分割线------------------
以上是博主目前能记得的Swift工程会遇到的恶心的坑，后续要是想起来其他的坑肯定会上来填坑，哎，做笔记有多重要，以后感觉要把自己遇到的每个bug，每个坑都记录下来，定期整理，搞不好还能出本书，哈哈~。

说完坑，我们再来说说Swift中 好的 让人激动人心的 Objective-C没有的 激起写代码欲望的（好想修饰词太多了）的好东西吧。当然不是讲语法，讲Swift语法好的网上一大坨一大坨的，这里要讲的是第三方库。

## RxSwift
玩过Objective-C的MVVM的同学肯定知道[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)这个库，吊炸天的一个库，目前Github上的Star已经破万了。那Swift上有没有这样的库呢，

当然有,那就是[RxSwift](https://github.com/ReactiveX/RxSwift)。
当然，ReactiveCocoa现在是4.0版本了，他在3.0版本的时候就出了Swift版的API，这里并不想讨论在Swift工程中那个库更好，stackoverflow上也有人全方位的比较过着两个库:[ReactiveCocoa vs RxSwift - pros and cons](http://stackoverflow.com/questions/32542846/reactivecocoa-vs-rxswift-pros-and-cons),看官们不妨可以移步过去看一下。

不过楼主最后还是打算使用RxSwift，毕竟用Swift实现，没有历史包袱，而且RxSwift的文档和Demo实在太全面太好了。而且RxSwift是[ReactiveX](https://github.com/ReactiveX)组织推出的，正宗王老吉，正宗好声音，不是，皇族血统，德玛西亚~

博主是在App发布后才了解到这个库，所以打算在App的下个版本接入RxSwift，到时候再分享接入心得给大家。

## Neon

[Neon](https://github.com/mamaral/Neon)是一个Swift上另辟蹊径的布局库。

Neon没有用AutoLayout来包装，AutoLayout我就不吐槽了，因为已经无力吐槽了。不过Swift上有个库叫做[Snapkit](https://github.com/SnapKit/SnapKit),是从OC的[Masonry](https://github.com/SnapKit/Masonry)演变过来的，现在Masonry也归属SnapKit组织名下了。不过Masonry的文档已经写的很清楚了：现在俺们只做bugfix以及合并一些高质量的PR，赶紧去看看Snapkit吧。可见Swift在国外俨然已成主流了。

说回Neon，他的语法类似描述，非常简单，可以有效减小我们布局代码的行数，不过个人感觉一行代码略长。当然博主也是从App发布后才知道这个库的，所以打算在App的下个版本接入Neon

## SQLite.swift

博主本来是用OC上老牌的[FMDB](https://github.com/ccgus/fmdb),但是当时FMDB接入Swift2.0工程一直报错，FMDB的作者也一直没怎么关心这件事，可能当时Swift2.0还是Bete的缘故。后来就接了SQLite.swift,总体来说蛮好用的，可能我的App本身存储的需求就比较简单，不过我问过SQLite.swift的作者，[SQLite.swift是没有缓存的](https://github.com/stephencelis/SQLite.swift/issues/234)，如果你要频繁操作一个数据库中的表数据的话，最好自己做一下缓存。


## SwiftyUserDefaults

操作`NSUserDefaults`很麻烦，每次都要写好几行代码。[SwiftyUserDefaults](https://github.com/radex/SwiftyUserDefaults)很好的利用了Swift的语法特性，让`NSUserDefaults`的操作达到了超级简单的水准，比如：
	
	Defaults[colorKey] = "red"
	Defaults[colorKey] // => "red", typed as String

恩，领先OC好几年~

## Ji
[Ji](https://github.com/honghaoz/Ji)是一个HTML/XML解析库，作者是一位加拿大华人，OC上也有类似的库[hpple](https://github.com/topfunky/hpple),当时作者接入的是hpple，后来发现了Ji，就试着接入Ji，然后发现这两个库有一个相同功能的API返回的结果不同，一问才知道是hpple的bug，果断抛弃hpple，投入Ji的怀抱。[那个bug链接](https://github.com/honghaoz/Ji/issues/6)


#####------------------我是第二条可爱的分割线------------------

好了，接下去我们来说说我的App.做iOS开发的都知道国内最大的苹果开发者技术资讯网站CocoaChina.com,可是这个网站却没有一个App，AppStore上是有一个官方的CocoaChina客户端app，但是已经好几年没有更新了，app里空无一物，一点数据都没有。AppStore上也有几个第三方开发者做的CocoaChina移动端的App，但是他们都有一个不好的地方就是没有做代码高亮，这样让我们看博客文章的时候很蛋疼，CocoaChina+就解决了这个问题：
##CocoaChina+

![iconpng](/images/custom/vender/icon.png)

先说一下CocoaChina+相对于市面上的app的几个亮点吧：

####1.代码高亮

目前市面上的第三方的CocoaChina的客户端app都没有做代码高亮，包括官方的Wap页面。这导致我们在手机端看博文的时候一到代码部分就非常蛋疼。CocoaChina+很好的解决了这个问题，极大的提高了阅读的体验。

####2.流量更省

文章渲染需要的CSS和JS代码CocoaChina+直接打包进了app内，每次文章加载的时候就不再需要去服务端获取一次了，极大的提高了加载速度，节省了用户的流量。

####3.纯黑设计

整个app采用纯黑色的设计，程序员都喜欢把自己的编辑器或者IDE界面调整成黑色，这样才可以把精力都集中在内容上，CocoaChina+的用户也基本都是程序员，因此也采用了纯黑色的设计，让用户在阅读文章的时候精力更加集中。

####4.内置聊天室

app内部整合了聊天室的功能，开发者可以直接进入和其他开发者直接匿名交流。是不是很好玩。

再说说这个app后续版本迭代需要更新的地方：

#####1.论坛

CocoaChina论坛由于有很多Apple的logo。所以目前App内的论坛都把图片去掉了，目前上线的是一个简单的论坛功能，后续会着力更新论坛模块，CocoaChina的论坛做的还是很牛B的，所以后续一定会有一个很nice的论坛模块展示在CocoaChina+中

#####2.登陆
	
CocoaChina+目前没有登录功能，导致目前论坛上大家还不能直接评论，这个后续也会更新维护

#####3.聊天界面

CocoaChina+的聊天功能是整合了第三方的UI，不是很nice，后续楼主会自己用Swift重写一套简洁的聊天UI更新上去

#####3.技术层面

CocoaChina+是一个纯Swift2.0的项目，用的第三方库也是能有Swift的就用Swift，最后才会考虑Objective-C。后续也会对代码做一次重构，整合进RxSwift(Swift版的ReactiveCocoa)和Swit上的布局框架Neon。

#####4.iPad版本

目前app只支持iPhone客户端，后续会推出iPad客户端

最后附上App安装二维码

![qrcode.png](/images/custom/vender/qrcode.png)

以及部分截图：

![home_cocoachina.jpg](/images/custom/vender/home_cocoachina.jpg)
![article_cocoachina.jpg](/images/custom/vender/article_cocoachina.jpg)
![code_cocoachina.jpg](/images/custom/vender/code_cocoachina.jpg)

##关于项目开源

这个App是纯Swift2.0编写的，目前项目还有很多没有上线的功能，部分功能还需要改善，代码也还需要整理，因此还不打算开源。不过等所有功能都上线了，楼主会整理下代码后开源到Github。到时候也会在App内发Push推送周知大家.不过这个过程可能会比较漫长，毕竟是个人项目，只能抽业余时间来做，还请大家耐心等待~

整个app整合了很多第三方平台，如友盟，极光推送，Google-Admob，环信IM等，对于今后有想做Swift项目的同学有很大的参考价值。

**希望有一天CocoaChina+会成为一个iOS开发者共同维护的App！**