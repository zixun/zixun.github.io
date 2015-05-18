---
layout: post
title: "[iOS单元测试系列]-译-OCMock常见使用方式"
date: 2015-04-16 14:51:27 +0800
comments: true
categories: 
---
**NOTE:** 本文翻译自OCMock官网中的一片教程外链[OCMock Test Origami](http://hackazach.net/code/2014/03/03/effective-testing-with-ocmock/)。

**该文章使用的API是OCMock老版本的API，新版本也兼容老版本的API，译者在用到老版本的API处已经添加了对应的新版本（OCMock3）的API供读者参考。**


###爱好者
这篇文章假设读者都能熟悉使用Xcode5的测试框架XCTest，或者BBD测试工具Kiwi或其他的iOS测试框架

###什么是mock？差不多就是纸老虎
当我们写单元测试的时候，不可避免的要去尽可能少的实例化一些具体的组件来保持测试既短又快。而且保持单元的隔离。在现代的面向对象系统中，测试的组件很可能会有几个依赖的对象。我们用mock来替代实例化具体的依赖class。mock是在测试中的一个伪造的有预定义行为的具体对象的替身对象。被测试的组件不知道其中的差异！你的组件是在一个更大的系统中被设计的，你可以很有信心的用mock来测试你的组件。

<!-- more -->

###常见的mock使用案例

####stub方法
我们用一个简单的例子来开始解释OCMock中一般的stub语法。
```objective-c
id jalopy = [OCMock mockForClass[Car class]];
 [[[jalopy stub] andReturn:@"75kph"] goFaster:[OCMArg any] units:@"kph"];
 // if returning a scalar value, andReturnValue: can be used
```

**OCMock3 新版本对应API**

```objective-c
id jalopy = OCMStrictClassMock([Car class]);
OCMStub([jalopy goFaster:[OCMArg any] units:@"kph"]).andReturn(@"75kph");
// if returning a scalar value, andReturnValue: can be used
```

这个简单的例子首先从Car类中mock出一个jalopy(老爷车),然后，stub掉`goFaster:`方法让它返回字符串`@”75kph”`。stub语法可能看起来有点奇怪，但这是普遍的做法：

`ourMockObject stub] whatItShouldReturn ] method:`

**OCMock3 新版本对应API**

`OCMStub([ourMockObject method:]).andReturn()`

一个非常重要的说明：注意`[OCMArg any]`的用法。当指定一个带参数的方法时，方法被调用并且参数为指定参数的话，mock会返回andReturn:指定的值。`[OCMArg any]`方法告诉stub匹配所有的参数值。举个例子：

`[car goFaster:84 units:@"mph"];`

不会触发stub，因为最后一个参数不匹配"kph".

####类方法
OCMock会在mock实例上没有找到相同名字的实例方法的时候去找同名的类方法。在名字相同的情况下（类方法和实例方法同名），用`classMethod`来指定类方法：

`[[[[jalopy stub] classMethod] andReturn:@"expired"] checkWarrany];`

**在OCMock3中classMethod和instanceMethod的stub方式一样，例如：**
```objective-c
id classMock = OCMClassMock([SomeClass class]);
OCMStub([classMock aClassMethod]).andReturn(@"Test string");
// result is @"Test string"
NSString *result = [SomeClass aClassMethod];
```

####mock类型 - niceMock,partialMock
OCMock提供了几种不同类型的mock，每个都有他们特定的使用场景。

用这种方式来创建任意mock：

`id mockThing = [OCMock mockForClass[Thing class]];`

**OCMock3 新版本对应API**

`id mockThing = OCMStrictClassMock([Thing class]);`

这就是我所说的`‘vanilla’ mock`。`‘vanilla’ mock`当调用一个没有stub的方法的时候会抛出一个异常。这会得到一个单调的mock，且在mock的生命周期中每一个方法调用都要被stub掉。(更多信息请看下一节关于stub)

如果你不想stub很多方法，用`‘nice’ mock`。`‘nice’ mock`非常有礼貌而且不会在一个没有stub掉的方法被调用的时候抛出异常。

`id niceMockThing = [OCMock niceMockForClass[Thing class]];`

**OCMock3 新版本对应API**

`id mockThing = OCMClassMock([Thing class]);`

最后一个mock类型是`‘partial’ mock`。当一个没有stub掉的方法被调用了，这个方法会被转发到真实的对象上。这是对mock技术上的欺骗，但是非常有用，当有一些类不适合让自己很好的被stub。

```objective-c
Thing *someThing = [Thing alloc] init];
id aMock = [OCMockObject partialMockForObject:someThing]
```

**OCMock3 新版本对应API**

```objective-c
Thing *someThing = [Thing alloc] init];
id aMock = OCMPartialMock(someThing);
```


####验证方法是否被调用

验证方法是否被调用非常简单。这个可以用`expect`来完成拒绝和验证方法:

```objective-c
id niceMockThing = [OCMock niceMockForClass[Thing class]];
 [[niceMockThing expect] greeting:@"hello"];
 // verify the method was called as expected
 [niceMocking verify];
```

**OCMock3 新版本对应API**

```objective-c
id niceMockThing = OCMClassMock([Thing class]);
OCMVerify([niceMockThing greeting:@"hello"]);
```



当被验证的方法没有被调用的时候会抛出异常。如果你用的是XCTest，那么请用`XCTAssertNotThrow`来包装验证调用。拒绝方法调用也是同样的道理，但是会再方法调用的时候抛出异常。就像stub，selector和传递过去验证的参数必须匹配调用时候传递过去的参数。用`[OCMArg any]`可以简化我们的工作。

####处理block参数

OCMock也可以处理block回调参数。block回调通常用于网络代码，数据库代码，或者在任何异步操作中。在这个例子中，思考下下面的方法：

```objective-c
- (void)downloadWeatherDataForZip:(NSString *)zip
              callback:(void (^)(NSDictionary *response))callback;
```
在这个例子中，我们有一个下载天气压缩数据的方法，并且把下载下来的dictionary代理到一个block的回调中。在测试中，我们通过预定义的天气数据来测试回调处理。这也是明智的测试失败场景。你永远不会知道网络上会返回你什么东西！

```objective-c
// 1. stub using OCMock andDo: operator.
[[[groupModelMock stub] andDo:^(NSInvocation *invoke) {
        //2. declare a block with same signature
        void (^weatherStubResponse)(NSDictionary *dict);
        //3. link argument 3 with with our block callback
        [invoke getArgument:&weatherStubResponse atIndex:3];
        //4. invoke block with pre-defined input
        NSDictionary *testResponse = @{@"high": 43 , @"low": 12};
        weatherStubResponse(groupMemberMock); 
    }]downloadWeatherDataForZip@"80304" callback:[OCMArg any] ];
```

**OCMock3 新版本对应API**

```objective-c
// 1. stub using OCMock andDo: operator.
OCMStub([groupModelMock downloadWeatherDataForZip:@"80304" callback:[OCMArg any]]]).andDo(^(NSInvocation *invocation){
        //2. declare a block with same signature
        void (^weatherStubResponse)(NSDictionary *dict);
        //3. link argument 3 with with our block callback
        [invoke getArgument:&weatherStubResponse atIndex:3];
        //4. invoke block with pre-defined input
        NSDictionary *testResponse = @{@"high": 43 , @"low": 12};
        weatherStubResponse(groupMemberMock); 
    });
```

这里的大体思想相当简单，即便如此，他的实现也需要一些说明：

	1.这个mock对象使用带NSInvocation参数的“andDo”方法。一个NSInvocation对象代表一个‘objectivetified’（实在不知道这个什么鬼）表现的方法调用。通过这个NSinvocation对象，使得拦截传递给我们的方法的block参数变得可能。
	2.用与我们测试的方法中相同的方法签名声明一个block参数。
	3.NSInvocation实例方法"getArgument:atIndex:"将赋值后的块函数传递都原始函数中定义的块函数中。注意：在Objective-C中，传递给任意方法的前两个参数都是“self”和“_cmd”.这是一个运行时的小功能以及用下标来获取NSInvocation参数时我们需要考虑的东西。
	4.最后，传递这个回调的预定义字典。
	
####最后
希望这篇文章和例子已经陈述清楚一些OCMock最通用的用法。OCMock站点：[http://ocmock.org/features/](http://ocmock.org/features/)是一个最好的学习OCMock的地方。mock是单调的但是对于一个现代的OO系统却是必须的。如果一个依赖图很难用mock来测试，这个迹象表明你的设计需要重新考虑了。