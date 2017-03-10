---
layout: post
title: "GodEye之日志监控"
date: 2017-03-07 16:27:04 +0800
comments: true
categories: 
---

日志几乎是我们每一个iOS开发者每一天都要打交道的东西，比如运行时想看一下某个变量的值，那就`NSLog()`一下;当然，在Swift语言下我们还有另外一种选择,那就是`print()`方法。都是帮助我们方便调试与分析的工具。

<!-- more -->

开源社区也为我们贡献了很多非常优秀的日志框架，比如OC中大名鼎鼎的`CocoaLumberjack`，有兴趣的同学可以移步[https://github.com/CocoaLumberjack/CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack)

开源日志框架是为了我们方便极了日志而开发的，比如我们可以调用CocoaLumberjack的`DDLogInfo("Info")`来记录一个`Info`类型的日志，用`DDLogError("Error")`来记录一个`Error`类型的日志。不同的日志记录框架我们需要调用的API也不一样，但是他们都有一个共同的特点，就是到最后都会调用系统的`NSLog()`方法。

因此我们需要做日志全自动监控的话，可以从`NSLog()`方法入手。这里我们会分两方面来讲，因为在iOS上我们不单单可以用`NSLog()`来记录日志，在Swift语言下还可以用`print()`方法来记录我们的日志。这一章我们会先介绍如何选择和优化我们监控`NSLog()`日志的方案，然后比较`NSLog()`与`print()`的不同点。然后再介绍Swift下特有的`print()`方法的处理。

# 0 前言
该文章摘自笔者撰写的一本iOS技术书籍[iOS监控编程](https://www.qingdan.us/product/25)。

这是一本介绍iOS监控编程的书籍,内容涉及日志监控，监控崩溃，监控网络，监控卡顿，监控硬件，内存泄漏监控等方面。所有的功能都是通过自行编程实现，而不是通过使用第三方工具。每个章节记录了功能的实现细节，以及笔者一路探索过来的心路历程。当然，笔者后续依旧会寻求与探索新的监控方向，一旦有所得都会更新到本书的章节中。

这本书是笔者的开源工具[GodEye](https://github.com/zixun/GodEye)的产出物，GodEye能全自动，零代码入侵，一行代码接入来监控应用的日志，卡顿，崩溃，网络，内存泄漏，CPU以及内存使用率，帧率等信息。

![hello world](/images/custom/post/2017-03-07-godeyezhi-ri-zhi-jian-kong/cover.jpg)


# 1 监控NSLog日志
该章节涉及到的开源库`ASLEye`,大家可以先通过[https://github.com/zixun/ASLEye](https://github.com/zixun/ASLEye)下载下来，一遍对照开源代码一遍阅读该章节。

可能你在翻开该章节前已经有了自己的方案了，因为这道题并没有唯一解，我们只是要找出最优解。下面让我们来逐一分析监控NSLog日志的方案。
## 自定义API
可能你和我一样，第一个想到的就是模仿`CocoaLumberjack`，咱整一套自己的API，让大家都来调咱的API不完事了么，多省事。比如`LogEyeInfo("Info")`就是`Info`类型的日志，`LogEyeError("Error")`就是`Error`类型的日志。

哈哈，咱是省事了，但是接入咱日志监控的开发者不干了，或许他正在和CocoaLumberjack愉快的玩耍，你侬我侬，你突然让我全换了，你他妈的在逗我？

![hello world](/images/custom/post/2017-03-07-godeyezhi-ri-zhi-jian-kong/你他妈的在逗我.jpg)

的确，这样接入方的开发者的工作量就太大了，而且也和全自动记录日志沾不上边，不是我们的最佳方案，换！

<br/>

## NSLog宏
相信大家这个时候也都想到了NSLog宏的方案。咱可以定义一个`NSLog`的宏，去覆盖掉系统的`NSLog`的方法。在咱的`NSLog`的宏里面去调用我们第一种方案中的自定义API就好。比如

```objective-c
#define NSLog(frmt, ...) AppInfo(frmt, ##__VA_ARGS__)
```
用这招偷梁换柱的方法，我们就可以在我们的`AppInfo`里面撒欢的处理我们的日志啦，用户一调我们就知道，用户一调我们就知道，好开心啊。赶紧找我们的接入方开发者来接入。

“我不接!”

“为啥子哟？”

“因为我现在用的开源日志记录框架也有了一个`NSLog`宏，把你的接入了日志记录框架的`NSLog`宏会被你覆盖掉，也有可能你的宏会被他的宏覆盖掉，谁先加载谁被覆盖！”

“......(握草泥马)”

![hello world](/images/custom/post/2017-03-07-godeyezhi-ri-zhi-jian-kong/握草泥马.jpg)


看来，`NSLog`宏的方案虽然好，但是一旦出现另一个`NSLog`宏就会出现“一山不容二宏”，“宏和宏不可兼得”的现象。我们不能保证接入方的开发者们不会接入其他编写了`NSLog`宏的第三方库，也不能保证接入方的开发者自己不会写一个`NSLog`宏。由此可见，`NSLog`宏的方案虽好，但是存在易冲突，不可靠，不可控等问题。换！

## 寻找新的方案
现在，两个方案都被我们排除了，还有啥比这两个更好的方案可以用呢。如果脑海里暂时没有好的方案与想法，我们就应该祭出我们程序员的基本生存技能，问度娘。。。。。。不是，问Google！

首先，我们去搜索一下Apple关于`NSLog`的文档。我们会发现`NSLog`方法中调用的是`NSLogv`方法。Apple文档中对于`NSLogv`函数是这样写的`Logs an error message to the Apple System Log facility`(参见[https://developer.apple.com/reference/foundation/1395074-nslogv?language=objc](https://developer.apple.com/reference/foundation/1395074-nslogv?language=objc))。

这句话有两个信息我们需要重点关注下，一个就是`error message`,还有一个就是`Apple System Log`,为啥`NSLog`是记录错误信息的，而不是日志信息呢？`Apple System Log`又是个什么鬼？

不急，我们再搜索下`CocoaLumberjack`的文档，我们会发现`CocoaLumberjack`的文档说了这么两句话:

    NSLog does 2 things:
        It writes log messages to the Apple System Logging (asl) facility. This allows log messages to show up in Console.app.

        It also checks to see if the application's stderr stream is going to a terminal (such as when the application is being run via Xcode). If so it writes the log message to stderr (so that it shows up in the Xcode console).
参见[https://github.com/CocoaLumberjack/CocoaLumberjack/blob/b1a3837366bc286ee24671ef1042b7214a8aa0ca/Documentation/Performance.md](https://github.com/CocoaLumberjack/CocoaLumberjack/blob/b1a3837366bc286ee24671ef1042b7214a8aa0ca/Documentation/Performance.md)

`CocoaLumberjack`的文档告诉我们，当我们写了一句`NSLog`的时候，他会做两件事情：1.把日志写到`Apple System Log`里面。2.把日志展示到Xcode的控制台上面。

两个文档都提到了`Apple System Log`，那这个`Apple System Log`到底是个什么鬼。其实`Apple System Log`可以理解为就是一个我们设备的日志数据库，这里存放了所有应用所有进程产生的日志。也就是说，不管在哪，只要你调用了`NSLog`,系统就会把它写到`Apple System Log`中。

那为啥`NSLog`是记录错误信息的，而不是日志信息呢？其实Apple设计`NSLog`的时候就是一个记录错误日志的API，不然他也不会把日志写到一个叫`Apple System Log`的数据库里面，要知道记录日志是一个高频发的操作，如果每条都放到数据库中，是一件很浪费性能的事情。所以我觉得应该叫`NSLogError`才更合适。

`NSLog`耗性能的一个原因也是因为需要把日志数据写到`Apple System Log`数据库，所以大伙的App线上Release版本尽量不要写`NSLog`，不但耗性能，而且不安全。Swift的`print`方法就不会把日志写到数据库中。当然在Swift中并没有废弃掉`NSLog`方法，但是Swift对`NSLog`方法做了优化，只有在模拟器环境下才会将日志写入`Apple System Log`。

## Apple System Log
OK，我们找到了我们的方案:`Apple System Log`。Apple为`Apple System Log`提供了一个framework供大家使用来对`Apple System Log`的数据进行操作。他就是`asl`,那下面我们就来试试怎么通过代码把`Apple System Log`的数据捞出来。

首先，我们需要把我们的`asl`模块import进来:

```swift
import asl
```

然后我们创建一个专门用来捞ASL的日志的类`ASLEye`,并且做个`Timer`定时器，定时拉取ASL的数据:

```swift
public class ASLEye: NSObject {

    public func open(with interval:TimeInterval) {
        self.timer = Timer.scheduledTimer(timeInterval: interval,target: self,selector: #selector(ASLEye.pollingLogs),userInfo: nil,repeats: true)
    }
    
    public func close() {
        self.timer?.invalidate()
        self.timer = nil
    }
 
    private var timer: Timer?
}
```
好了，现在就让我们来实现`pollingLogs`方法，来拉取我们的数据。
```swift
@objc private func pollingLogs() {
    //TODO  调用拉取数据API，获得数据数组
    //TODO  判断数据数组是否为空
    //TODO  将数据数组回调给上层
    }
```
咱先不着急`pollingLogs`方法的具体实现。大家可以和我一样先注释好方法要做的事情。

这里我们要先介绍下`asl`的api的特点。`asl`的api类似SQL语句，你可以往查询语句里面加参数来过滤数据，比如要查询当前App的日志的话，需要设置App的BundleID，当然，我们每次启动我们的App的时候都是不同的进程，所以我们还需要设置当前App的进程ID。因此我们可以编写一个`initQuery`的方法来生成我们的查询语句：

```swift
private func initQuery() -> aslmsg {
        var query: aslmsg = asl_new(UInt32(ASL_TYPE_QUERY))
        //set BundleIdentifier to ASL_KEY_FACILITY
        let bundleIdentifier = (Bundle.main.bundleIdentifier! as NSString).utf8String
        asl_set_query(query, ASL_KEY_FACILITY, bundleIdentifier, UInt32(ASL_QUERY_OP_EQUAL))
        
        //set pid to ASL_KEY_PID
        let pid = NSString(format: "%d", getpid()).cString(using: String.Encoding.utf8.rawValue)
        asl_set_query(query, ASL_KEY_PID, pid, UInt32(ASL_QUERY_OP_NUMERIC))
        
        return query
    }
```

然后我们来编写拉取数据API，获得数据数组的API，这个API要做的事情就是调用`initQuery()`方法生成我们的查询语句，然后解析返回的response，将response解析成一个字符串的数组返回给调动着，咱就叫他`retrieveLogs()`方法：
```swift
private func retrieveLogs() -> [String] {
        var logs = [String]()
        
        var query: aslmsg = self.initQuery()
        
        let response: aslresponse? = asl_search(nil, query)
        guard response != nil else {
            return logs
        }
        
        var message = asl_next(response)
        while (message != nil) {
            let log = self.log(from: message!)
            logs.append(log)
            
            message = asl_next(response)
        }
        asl_free(response)
        asl_free(query)
        
        return logs
    }

private func parserLog(from message:aslmsg) ->String {
        let content = asl_get(message, ASL_KEY_MSG)!;
        return String(cString: content, encoding: String.Encoding.utf8)!
    }
```

好，现在我们让`pollingLogs()`来调用我们的`retrieveLogs()`方法：
```swift
@objc private func pollingLogs() {
        let logs = self.retrieveLogs()
        if logs.count > 0 {
            self.delegate?.aslMonitor?(aslMonitor: self, didMonitorLogs: logs)
        }
    }
```
这样我们的日志就能通过我们的delegate方法回调给上层了。当然在这之前我们需要编写一个我们这个delegate的`protocol`：
```swift
@objc public protocol ASLEyeDelegate: class {
    @objc optional func aslEye(aslEye:ASLEye,catchLogs logs:[String])
}
```
然后在`ASLEye`上添加一个delegate的变量:
```swift
    public weak var delegate: ASLEyeDelegate?
```

但是，这个时候大家去调用NSLog会发现存在一个问题。比如我调用了下`NSLog("Hello")`,我们的日志监控每次轮询都会取到我们的日志数据，我们的delegate会一直回调，把`Hello`给上层。

会造成这样其实也不难理解，因为App System Log是一个数据库，按照咱现在的查询语句，每次插叙的时候当然会把当前App当前的日志全部返回给我们，所以，我们还需要设置一个'游标'，换句话说就是我们需要在我们的查询语句里加一个参数，将我们上次查到的最后一个日志的id给塞进去，然后查询大于这个ID的日志。

OK,首先，我们来设置一个变量`lastMessageID`，用来记录最后一次查到的日志ID：
```swift
private var lastMessageID: Int32 = 0
```
然后在我们的`initQuery()`方法里最后返回我们的query前将我们的`lastMessageID`给塞进去：
```swift
private func initQuery() -> aslmsg {
        var query: aslmsg = asl_new(UInt32(ASL_TYPE_QUERY))
        //set BundleIdentifier to ASL_KEY_FACILITY
        let bundleIdentifier = (Bundle.main.bundleIdentifier! as NSString).utf8String
        asl_set_query(query, ASL_KEY_FACILITY, bundleIdentifier, UInt32(ASL_QUERY_OP_EQUAL))
        
        //set pid to ASL_KEY_PID
        let pid = NSString(format: "%d", getpid()).cString(using: String.Encoding.utf8.rawValue)
        asl_set_query(query, ASL_KEY_PID, pid, UInt32(ASL_QUERY_OP_NUMERIC))
        
        if self.lastMessageID != 0 {
            let m = NSString(format: "%d", self.lastMessageID).utf8String
            asl_set_query(query, ASL_KEY_MSG_ID, m, UInt32(ASL_QUERY_OP_GREATER | ASL_QUERY_OP_NUMERIC));
        }
        
        return query
    }
```

最后，我们需要在每次解析我们的日志的时候将我们的日志ID给记录下来放到`lastMessageID`中：
```swift
private func parserLog(from message:aslmsg) ->String {
        let content = asl_get(message, ASL_KEY_MSG)!;
        let msg_id = asl_get(message, ASL_KEY_MSG_ID);
        
        let m = atoi(msg_id);
        if (m != 0) {
            self.lastMessageID = m;
        }
        
        return String(cString: content, encoding: String.Encoding.utf8)!
    }
```
OK,这样，我们就能够全自动的监控我们的日志啦。上层开发者还是按照他们的方式玩就好，我们都能拿到他们的日志。
该章节涉及到的开源库`ASLEye`有兴趣的同学可以移步[https://github.com/zixun/ASLEye](https://github.com/zixun/ASLEye)查看。

<br />
<br />

## 更好的方案
通过ASL来监控NSLog日志虽然能很好的完成自动监控的功能，但是毕竟`Apple System Log`是一个数据库，每次做查询操作由于会产生I/0操作，所以会产生一定的性能消耗。虽然这点性能消耗在日常应用的运行时完全可以忽略不计，但是当我们在统计应用整个流程的CPU使用率的时候还是会造成一定的干扰，那有没有更好的方案呢？

当然有。那就是将`NSLog`方法给hook掉。不过当我们点开`NSLog`方法的时候可以发现这个API并不是一个Objective-C语言的方法，而是一个C语言包装的API。C语言的API当然不能用我们Runtime的Method Swizzle来替换，他并不会走OC的运行时。

不过方法总比问题多，Facebook有一个名字叫fishhook的开源库，就可以做到将我们的`Mach-O`的库的符号表重新绑定，因此应该也能将我们的`NSLog`重新绑定，重新绑定后就可以做我们的日志监控啦。

有兴趣的同学可以自己实验下，笔者也会在今后`GodEye`的更新迭代中探索替换该方案来实现全自动的日志监控。当然也会将实现原理编写成该书的一个章节提供给大家。

# 1 监控print日志

前一章节我们分析了`NSLog`日志的监控，不过接触过Swift语言的同学都知道,Swift中大家一般都是用`print`来记录日志。当然，Swift并没有将`NSLog`给废弃掉。

## NSLog In Swift
当然，Swift中我们也可以用`NSLog`来做日志记录，但是Swift对`NSLog`做过优化。笔者发现Swift中调用`NSLog`方法，它并不会想Objective-C中调用`NSLog`方法那样将日志放入`Apple System Log`中，模拟器环境下除外。也就是说我们在真实环境下并不能通过`ALS`获取`NSLog`的日志信息。因此我们需要寻找其他的办法。

##自定义print方法
我们来看看Swift中`print`方法的定义：

```swift
public func print(_ items: Any..., separator: String = default, terminator: String = default)
```
其中`separator`的默认值是一个" "（空格字符串）。`terminator`的默认值是"\n"（回车符）。

我们自定义的`print`方法不能完全和Swift的一样,不然编译器会给你一个报错`Ambiguous use of 'print(_:separator:terminator:)'`,因为两个API一模一样，编译器就不知道去链接哪个API了。

不过我们可以把默认值去掉，只需要一个item参数就好：

<br />

```swift
public func print(_ items: Any...) {
    //TODO: Custom Actions
    Swift.print(items)
}
```
这样，我们只需要在使用print方法的swift文件中，import我们这个自定义api的库就好了。然后将`TODO: Custom Actions`中放入我们的日志监控操作即可。

##自定义日志框架
自定义print方法固然不错，但是使用者很容易忘了import我们这个自定义api的库，Swift中也没有预加载的功能，因此使用者很容易忘记做import操作，导致监控不到。而且单独一个print方法无法优雅的区分日志的类型，比如log,warning,error等。更不能获取文件名，方法名等日志辅助信息。因此我们何尝不自定义一个轻量级的日志框架呢。

该日志框架大家可通过[https://github.com/zixun/Log4G](https://github.com/zixun/Log4G)下载，文中由于篇幅的原因，只会列出关键代码，建议将开源源码下载后对照着源码查看，会更便于理解。

我们先来看一张`Log4G`的简要设计图：

![hello world](/images/custom/post/2017-03-07-godeyezhi-ri-zhi-jian-kong/log4g_design.jpg)

从图上我们可以看出，我们对外有三个接口，分别是`log()`,`warning()`和`error()`。他们内部都会调用一个不公开的接口`record()`,当然在方法中会自动获取日志的文件（file），行数（line），方法名（function）以及线程（thread），然后传递给`record()`方法，`record()`方法内部所有的操作都在一个指定的queue中进行，可以防止多线程的问题。`record()`会做两件事，一个就是调用系统的`print()`函数，还有一个就是通知`Log4g`内部维护的delegate数组，告诉日志监听者们新的日志信息。

我们先来看看`log()`的实现：

```swift
/// record a log type message
///
/// - Parameters:
///   - message: log message
///   - file: file which call the api
///   - line: line number at file which call the api
///   - function: function name which call the api
open class func log(_ message: Any = "",
                    file: String = #file,
                    line: Int = #line,
                    function: String = #function) {
    self.shared.record(type: .log,
                       thread: Thread.current,
                       message: "\(message)",
                       file: file,
                       line: line,
                       function: function)
}
```
我们通过`#file`,`#line`,`#function`可以自动获取调用API时候的文件名，行数以及方法名，不需要调用者参与。然后通过`Thread.current`拿到当前线程，一并传递给`record`方法。这样调用者只需要传递日志即可，其他一切都是自动。

`warning()`和`error()`原理一致，那我们来看看`record()`方法：

```swift
/// record message base function
///
/// - Parameters:
///   - type: log type
///   - thread: thread which log the messsage
///   - message: log message
///   - file: file which call the api
///   - line: line number at file which call the api
///   - function: function name which call the api
private func record(type:Log4gType,
                 thread:Thread,
                 message: String,
                 file: String,
                 line: Int,
                 function: String) {
    self.queue.async {
        let model = LogModel(type: type,
                             thread: thread,
                             message: message,
                             file: self.name(of: file),
                             line: line,
                             function: function)
        print(message)
        
        for delegate in self.delegates {
            delegate.delegate?.log4gDidRecord(with: model)
        }
        
    }
}
```
`record()`方法内部通过`self.queue.async{...}`将所有的操作都同步在一个统一的线程中，然后调用系统的`print()`方法并生成一个日志的model，告诉`Log4g`内部维护的delegate数组监听到的新的日志。这里涉及成delegate数组可以方便日志监听的扩展，`Log4g`顾名思义，就是`Log4GodEye`,就是设计给`GodEye`使用的日志记录框架，`GodEye`是delegate的一员，但是可能用户的App其他地方也需要监控日志，只要将其加入到delegate即可。

附`GodEye`开源地址：[https://github.com/zixun/GodEye](https://github.com/zixun/GodEye)，`GodEye`是一个可以自动监控日志，崩溃，卡顿，网络，内存泄漏，沙盒文件，CPU,RAM，FPS,流量等信息的工具，功能丰富，just like god open his eye。

##delegate数组的坑
在Objective-C中，我们将delegate放入一个数组中完全没有问题，因为`NSArray`不会强引用放进来的对象。但是Swift中的`Array`是是一个结构体，会强引用放入的元素。比如我们的delegate是一个ViewController，将我们的ViewController放入Array后，只要Array不释放，我们的ViewController就永远不会释放，但是ViewController可能已经被pop出去了，这就造成了内存泄漏。那怎么办呢？

只要做一层weak化的包装就好，就拿`Log4g`来说，内部维护了一个delegate的数组：

```swift
fileprivate var delegates = [WeakLog4GDelegate]()
```
那`WeakLog4GDelegate`是个什么鬼呢，这个属性的申明是`fileprivate`的，说明用户是不知道`WeakLog4GDelegate`的，我们来看看`WeakLog4GDelegate`：

```swift
class WeakLog4GDelegate: NSObject {
    weak var delegate : Log4GDelegate?
    init (delegate: Log4GDelegate) {
        super.init()
        self.delegate = delegate
    }
}
```
可以看到，`WeakLog4GDelegate`是`Log4GDelegate`的一个weak化包装，内部有一个`Log4GDelegate`的weak属性，也通过`Log4GDelegate`来初始化。

这样当我们将`WeakLog4GDelegate`放入数组后，外部的Log4GDelegate就不会因为强制引用而无法释放了。当然`WeakLog4GDelegate`并不是一个`open`或者`public`的类，所以我们还需要包装一些方法让使用者添加或删除外部使用的`Log4GDelegate`：

```swift
open class func add(delegate:Log4GDelegate) {
    let log4g = self.shared
    // delete null week delegate
    log4g.delegates = log4g.delegates.filter {
        return $0.delegate != nil
    }
    // judge if contains the delegate from parameter
    let contains = log4g.delegates.contains {
        return $0.delegate?.hash == delegate.hash
    }
    // if not contains, append it with weak wrapped
    if contains == false {
        let week = WeakLog4GDelegate(delegate: delegate)
        self.shared.delegates.append(week)
    }
}
open class func remove(delegate:Log4GDelegate) {
    let log4g = self.shared
    log4g.delegates = log4g.delegates.filter {
        // filter null weak delegate
        return $0.delegate != nil
    }.filter {
        // filter the delegate from parameter
        return $0.delegate?.hash != delegate.hash
    }
}
```
