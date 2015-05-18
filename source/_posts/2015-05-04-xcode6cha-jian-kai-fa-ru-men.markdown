---
layout: post
title: "Xcode6插件开发入门"
date: 2015-05-04 17:34:03 +0800
comments: true
categories: 
---
Xcode提供了所有你创建一个App需要的功能。但是由于其不开源以及没有制作Xcode-Plugin相关的文档，在我们需要添加一些自己的想法和功能的时候变得缺乏灵活性。
但是我们可以通过一些非官方的手段来扩展我们自己的Xcode，并且分享给别人使用。

<!-- more -->

##Xcode Plug-in能做什么

太多了，我们可以自动生成代码注释（[VVDocumenter](https://github.com/onevcat/VVDocumenter-Xcode)）,我们可以在代码编辑器中直接显示我们初始化的UIColor的颜色（[ColorSense-for-Xcode](https://github.com/omz/ColorSense-for-Xcode)）,我们也可以在代码编辑器中直接显示我们要添加到UIImage的图片（[KSImageNamed-Xcode](https://github.com/ksuther/KSImageNamed-Xcode)）,我们还可以调整控制台的颜色，修改代码样式，等等等等....

我们还可以用[Alcatraz](https://github.com/supermarin/Alcatraz)来管理我们的插件，简直方便！

##Xcode Plug-in放在哪
所有的Xcode Plug-in都会放在一个叫`~/Library/Application Support/Developer/Shared/Xcode/Plug-ins`文件目录下，并且所有的插件都会以`.xcplugin`作为后缀。


##开发自己的Xcode Plug-in

###准备工作
	1.前往[Xcode-Plugin-Template](https://github.com/kattrali/Xcode-Plugin-Template)下载Xcode插件开发的模板。
	2.将下载下来的template复制到 ~/Library/Developer/Xcode/Templates/Project Templates/Application Plug-in/Xcode5 Plugin.xctemplate文件夹中，如果没有对应的文件夹就自己手工创建一个。
	3.重启Xcode，当你新建一个工程的时候就可以在OSX中看到一个Application Plug-in的选项，里面有一个Xcode Plug-in模板。

![hello world](/images/custom/post/xcode6cha-jian-kai-fa-ru-men/template.png)

###进入开发

我们新建一个Xcode Plug-in模板的工程，可以看到模板为我们生成了好多代码。看与你工程同名的文件的`initWithBundle`方法。阅读一下后你就可以知道模板为我们填充了一个在`Edit`中添加一个`Do Action`的按钮，点击后会回调`doMenuAction`方法。

我们可以run一下我们的工程，你会发现启动了一个新的Xcode，因为我们做的是Xcode插件，启动的当然是Xcode，用Xcode编写Xcode的代码，是不是很有意思。试试点击新的Xcode的Edit有一个`Do Action`按钮，点击`Do Action`按钮，再回调中打个断点，看有没有回调你的方法。

####DVTPlugInCompatibilityUUIDs

是的，你会发现你的回调方法木有调用，木有调用啊，什么鬼！其实这是因为你的插件不认识你正在跑的Xcode的版本，把你的Xcode版本介绍给它认识就好了。

	1.打开插件工程的Info.plist,找到DVTPlugInCompatibilityUUIDs，打开这个下拉框，这里有所有你的插件认识的Xcode版本。
	2.在terminal中输入 defaults read /Applications/Xcode.app/Contents/Info DVTPlugInCompatibilityUUID ，terminal会返回一串字符串给你，这就是你的Xcode的DVTPlugInCompatibilityUUID，把这串字符串添加到DVTPlugInCompatibilityUUIDs中即可。
	3.再跑一次Xcode，是不是可以了~

####窃听Xcode通知

因为Apple至今也没有公开Xcode Plugin的文档。所以我们需要通过一些其他的思路寻找方法。比如窃听我们在操作Xcode的时候Xcode发出的各种通知。

```objective-c
	[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(notificationLog:) name:nil object:nil];
```
将这行代码加入到我们的工程初始化中

```objective-c
- (void)notificationLog:(NSNotification *)notify
{
	NSLog(@"%@",notify.name); 
}
```
这样我们就可以监听我们在对xcode做操作的时候xcode发出的所有通知了。跑一下，是不是通知刷刷的来了。。。

###实现一个Xcode插件功能

####寻找需求
既然我们都知道了这么多通知，我们是不是可以通过抓取其中某一个通知动手优化下Xcode呢。现在Xcode插件辣么多，好多咱们能想到的功能，其实用心在网上google一下，都能找到。鉴于入门，那么我们来坐一个比较简单的功能吧。

比如我们在xcode文件导航区域要显示这个文件在Finder中的位置的时候，我们需要在这个文件中右键点击，然后在展示列表中点击`show in finder`，而且这个`show in finder`按钮还没有快捷键。要是能有个快捷键能够快速打开就好了。

####构思实现
	1.我们需要监听文件导航区域选中的文件改变时候的通知，然后保存当前选中的文件的路径
	2.我们需要新建一个NSMenuItem，然后设置我们需要的快捷键。
	3.当我们按下快捷键的时候打开我们的workspace，并且选中到指定的文件。

####talk is cheep，show u the code

1.我们需要监听文件导航区域选中的文件改变时候的通知，然后保存当前选中的文件的路径

根据之前监听xcode通知，我们会发现一个叫`transition from one file to another`这个通知，这就是我们需要的选中文件改变时候的通知。

```objective-c
- (void)notificationLog:(NSNotification *)notify
{
    if ([notify.name isEqualToString:@"transition from one file to another"]) {
        NSURL *originURL = [[notify.object valueForKey:@"next"] valueForKey:@"documentURL"];
        
        if (originURL != nil && [originURL absoluteString].length >= 7 ) {
            self.url = [originURL.absoluteString substringFromIndex:7];
        }
    }
}
```

我们将这个通知给我们的文件路径保存下来，为什么要`substringFromIndex:7`，因为返回的URL是`file:///User........`格式的，需要切换成`/User...`的格式。

2.我们需要新建一个NSMenuItem，然后设置我们需要的快捷键。

这个比较简单修改下模板帮我们生成的那几行代码，加入快捷键就可以
```objective-c
NSMenuItem *menuItem = [[NSApp mainMenu] itemWithTitle:@"Edit"];
        if (menuItem) {
            [[menuItem submenu] addItem:[NSMenuItem separatorItem]];
            NSMenuItem *actionMenuItem = [[NSMenuItem alloc] initWithTitle:@"Do action" action:@selector(doMenuAction) keyEquivalent:@"F"];
            [actionMenuItem setKeyEquivalentModifierMask:NSShiftKeyMask];
            [actionMenuItem setTarget:self];
            [[menuItem submenu] addItem:actionMenuItem];
        }
```objective-c

关键在于这两行：

```objective-c
NSMenuItem *actionMenuItem = [[NSMenuItem alloc] initWithTitle:@"Do action" action:@selector(doMenuAction) keyEquivalent:@"F"];
            [actionMenuItem setKeyEquivalentModifierMask:NSShiftKeyMask]; 
```
这里的快捷键就是`shift + F`，读者可以根据自己的方式修改。


3.当我们按下快捷键的时候打开我们的workspace，并且选中到指定的文件。
```objective-c
- (void)doMenuAction
{
    self.url =  [self.url stringByURLDecodingStringParameter];
    [[NSWorkspace sharedWorkspace] selectFile:self.url inFileViewerRootedAtPath:nil];
}
```

这个就比较好理解了。把第一步保存的URL传递过去就可以，不过需要做一个URLDecoding，因为xcode给我们的是encode过的URL。

##Xcode "API"
Xcode插件开发已经不是什么新技术了，所以也有很多开发者dump出了很多Xcode的API。大家可以按需要加入到自己的工程中使用。

###Working with projects files
[XcodeEditor](https://github.com/appsquickly/XcodeEditor)有一组强大的操纵项目文件的API，比如inspect headers, list frameworks, add classes等功能。如果你要对你项目文件进行操作，千万不要忘记这个库。

###NSTask
`NSTask`的功能类似像terminal中执行command命令一样，非常实用。

[nstask-tutorial](http://www.raywenderlich.com/36537/nstask-tutorial)

###Cocoa
是`Cocoa`，不是`Cocoa touch`，应为Xcode是MacOS上的app。所以编写Xcode插件会用到很多`Cocoa`的API，需要了解`Cocoa`编程。


##扩展阅读

[Xcode5 Plugins 开发简介](http://studentdeng.github.io/blog/2014/02/21/xcode-plugin-fun/)

[Creating an Xcode Plugin: A Quick-Start Guide](http://www.overacker.me/blog/2015/01/25/creating-an-xcode-plugin)

[Extending Xcode 6 with plugins](http://bits.citrusbyte.com/extending-xcode-6-with-plugins/)

[How to create Xcode plugin](http://www.fantageek.com/1297/how-to-create-xcode-plugin/)