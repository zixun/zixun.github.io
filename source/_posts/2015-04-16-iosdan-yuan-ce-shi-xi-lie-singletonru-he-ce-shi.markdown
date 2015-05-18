---
layout: post
title: "[iOS单元测试系列]Singleton如何测试"
date: 2015-04-16 14:53:44 +0800
comments: true
categories: 
---

Singletion设计模式在cocoa中被广泛使用。在我们平时写App代码时也经常会将一些工具类，管理类设计成Singletion。Signletion通过一个类方法返回一个唯一的实例，与我们平常通过实例化生成一个个实例的场景有所不同。如果我们要stub一个Singletion的类的实例方法，那么这个Signletion的类初始化方法（eg:sharedMange()）必须返回一个mock对象。因为只有mock对象才可以做stub操作。那么我们应该如何mock我们的Singletion呢，我们通过下面的例子一步步分析解决这个问题。

<!-- more -->

#Singleton场景

比如我有一个Singleton的类（DemoStatusManage）,他有一个实例方法currentStatus会返回一个1-100的随机数。

```objective-c
@interface DemoStatusManage : NSObject

+ (instancetype)sharedManage;

- (int)currentStatus;
@end

@implementation DemoStatusManage {
    NSInteger _status;
}

+ (instancetype)sharedManage {
    static dispatch_once_t once;
    static DemoStatusManage *manage;
    dispatch_once(&once, ^{
        manage = [[DemoStatusManage alloc] init];
    });
    return manage;
}

- (instancetype)init {
    self = [super init];
    if (self) {
        _status = 0;
    }
    return self;
}

- (int)currentStatus {
    return [self getRandomNumber:1 to:100];
}

-(int)getRandomNumber:(int)from to:(int)to {
    return (int)(from + (arc4random() % (to - from + 1))); 
}
@end
```

然后在我的另外一个类中会去调用这个Singletion的currentStatus方法，并且将返回的数据渲染到另外那个类的label文案上。

```objective-c
- (void)updateStatusNumber {
    self.statusLabel.text = [NSString stringWithFormat:@"%ld",(long)[[DemoStatusManage sharedManage] currentStatus]];
}
```
这是一个很简单的Singletion场景，但是在测试`updateStatusNumber`这个API的时候由于依赖到了外部的DemoStatusManage的currentStatus方法，而且这个方法返回的是一个随机数值，所以我们必须mock掉Singletion，然后再stub调currentStatus方法，让这个方法返回我们期望的一个固定值。

#应该用OCMock的哪个API呢

应该用OCMock的哪个API呢？OCMStrictClassMock(cls)? OCMClassMock(cls)? OCMPartialMock(obj)? 

其实这里按照常规的mock测试一个API都用不上。因为我们mock出来的东西（对象或者是类）只能在我们的测试用例中，updateStatusNumber方法里面调用的永远是DemoStatusManage的原生类。

那如何才能让sharedManage不管在哪里（测试用例中和updateStatusNumber中）都返回我们的mock对象呢，答案是用category重写sharedManage让它返回我们的mock对象.

```objective-c
@interface DemoStatusManage (UnitTest)

@end

static DemoStatusManage *mock = nil;
@implementation DemoStatusManage (UnitTest)

+ (instancetype)sharedManage {
    if (mock)  return mock;
    
    static dispatch_once_t once;
    static DemoStatusManage *manage;
    dispatch_once(&once, ^{
        manage = [[DemoStatusManage alloc] init];
    });
    return manage;
}

@end
```
这样在我们的单元测试类中只要在测试case中初始化一下mock，sharedManage不管在哪里调用就都会返回我们需要的mock对象了。

```objective-c
mock = OCMClassMock([DemoStatusManage class]);
```

当然我们也可以让mock返回一个PartialMock对象。

```objective-c
mock = OCMPartialMock([[DemoStatusManage alloc] init]);
```

#包装优化

###去掉拷贝的代码
你应该也发现了，这段代码我们是拷贝过来的。

```objective-c
   static dispatch_once_t once;
   static DemoStatusManage *manage;
   dispatch_once(&once, ^{
       manage = [[DemoStatusManage alloc] init];
   });
   return manage;
```
如果用这种方式，我们会陷入一个问题，我们在维护两套相同的代码，那天app工程中相关的sharedManage的方法有所变动，这里也要相应的变动。有什么办法可以让它找到原来的IMP实现呢，Matt大神的一篇文章中就告诉我们，Yes，可以的！[Supersequent implementation](http://www.cocoawithlove.com/2008/03/supersequent-implementation.html).我们可以用Matt的invokeSupersequentNoArgs()宏定义来实现这个功能。

这样我们的Cagegory差不多就长这样。
```objective-c
@interface DemoStatusManage (UnitTest)

@end

static DemoStatusManage *mock = nil;
@implementation DemoStatusManage (UnitTest)

+ (instancetype)sharedManage {
    if (mock)  return mock;
    
    return invokeSupersequentNoArgs()
}
@end
```

###包装mock方法

笔者在用这种方式写测试用例的时候发现，可能我的UnitTest这个Category是写在Atest.m中的，但是在没有写Category也没有引用Atest.m的Btest.m中，也会进入到重写的sharedManage中，而由于mock是static的，也没有做释放操作，导致DemoStatusManage永远是一个mock对象。可能是因为XCTest框架的原因，因为所有的XCTestCase都是没有.h文件的，具体原因也不得而知。

所以，要解决这个问题，我们必须在mock使用完毕后释放它，并且将创建和释放都包装出来，提供接口给测试用例调用。而且我们可以提供不同类型的mock方式。

```objective-c
@interface DemoStatusManage (UnitTest)

+ (instancetype)JTKCreateClassMock;

+ (instancetype)JTKCreatePartialMock:(DemoStatusManage *)obj;

+ (void)JTKReleaseMock;

@end

static DemoStatusManage *mock = nil;
@implementation DemoStatusManage (UnitTest)

+ (instancetype)sharedManage {
    if (mock)  return mock;
    
    return invokeSupersequentNoArgs();
}

+ (instancetype)JTKCreateClassMock {
    mock = OCMClassMock([DemoStatusManage class]);
    return mock;
}

+ (instancetype)JTKCreatePartialMock:(DemoStatusManage *)obj {
    mock = OCMPartialMock(obj);
    return mock;
}

+ (void)JTKReleaseMock {
    mock = nil;
}

@end
```

这样我们就可以在使用mock的时候调用JTKCreateClassMock 或者 JTKCreatePartialMock: 来生成我们需要的mock对象，在使用完毕后释放我们的mock对象，就能实现我们的测试需求了。

###宏定义简化代码

我们的工程中不可能只有一个Singletion，少则十几，多则上百。如果我们对每个Singletion都这么写一遍Category的话，这个成本也太他妈大了。而其实不管是哪个Singletion，这个UnitTest的Category都是大同小异的，那么我们不如写个宏定义来简化我们的代码。



```objective-c
#define JTKMOCK_SINGLETON(__className,__sharedMethod)               \
JTKMOCK_SINGLETON_CATEGORY_DECLARE(__className)                     \
JTKMOCK_SINGLETON_CATEGORY_IMPLEMENT(__className,__sharedMethod)    \


#define JTKMOCK_SINGLETON_CATEGORY_DECLARE(__className)         \
                                                                \
@interface __className (UnitTest)                               \
                                                                \
+ (instancetype)JTKCreateClassMock;                             \
                                                                \
+ (instancetype)JTKCreatePartialMock:(__className *)obj;        \
                                                                \
+ (void)JTKReleaseMock;                                         \
                                                                \
@end


#define JTKMOCK_SINGLETON_CATEGORY_IMPLEMENT(__className,__sharedMethod)    \
                                                                            \
static __className *mock_singleton_##__className = nil;                     \
                                                                            \
@implementation __className (UnitTest)                                      \
                                                                            \
+ (instancetype)__sharedMethod {                                            \
    if (mock_singleton_##__className) return mock_singleton_##__className;  \
    return JTKInvokeSupersequentNoParameters();                             \
}                                                                           \
+ (instancetype)JTKCreateClassMock {                                        \
    mock_singleton_##__className = OCMClassMock([__className class]);       \
    return mock_singleton_##__className;                                    \
}                                                                           \
                                                                            \
+ (instancetype)JTKCreatePartialMock:(__className *)obj {                   \
    mock_singleton_##__className = OCMPartialMock(obj);                     \
    return mock_singleton_##__className;                                    \
}                                                                           \
                                                                            \
+ (void)JTKReleaseMock {                                                    \
    mock_singleton_##__className = nil;                                     \
}                                                                           \
                                                                            \
@end
```
这样我们只需要一行代码就能搞定一个Singletion的UnitTest的Category了，来一个写一行，来一双写两行。

```objective-c
JTKMOCK_SINGLETON(DemoStatusManage,sharedManage)
```

#One more thing

Matt文中代码可以在github上找到[NSObject+SupersequentImplementation](https://github.com/tianyouhui/SimpleEncapsulate/blob/cb5ee50c69791b4f2ccabea06caa7a72a88f4354/SimpleEncapsulate/SimpleEncapsulateTests/OCMockSingletonExtension/NSObject%2BSupersequentImplementation.h)

如果使用`invokeSupersequentNoArgs()`提示`Too many arguments to function call,expected 0,have 2`,请打开你的测试工程的target，找到Build Setting下的`Enable Strict Checking of objc_mesSend Calls`,设置为NO

用category重写主类中的方法会有一个警告：`Category is implementing a method which will also be implemented by its primary class`,则使用以下宏在你重写的方法前后做个包装即可
```objective-c
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wobjc-protocol-method-implementation"

JTKMOCK_SINGLETON(DemoStatusManage,sharedManage)

#pragma clang diagnostic pop
```


