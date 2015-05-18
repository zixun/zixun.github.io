---
layout: post
title: "[iOS单元测试系列]单元测试编码规范"
date: 2015-04-16 18:11:16 +0800
comments: true
categories: 
---
编写单元测试与编写工程代码略有不同。我们需要准备数据，mock对象，调用工程Api,验证结果。而且一般测试代码都会比工程代码要大。就像[Real-World Testing with XCTest](http://www.objc.io/issue-15/xctest.html)一文中提到“目前为止，我们的编码库已经纵横 190 个文件和 18,000 行代码，达到了 544 kB。我们测试部分的代码现在差不多有1,200 kB，大概有被测试代码的两倍”。那么应该如何定义单元测试代码编写规范，使得代码更整洁，可读性更高呢？

<!-- more -->

**为了工程代码的保密，文中的代码在命名的时候在不影响理解的情况下都做了改动**

##1.Given-When-Then分段
每个case其实都可以分为三步走，1.mock对象，准备测试数据。2.调用目标API 3.验证输出和行为。所以我们可以用如下方式将3步分别放入Given-When-Then三个分段中。（为了保密，代码做了改动）

```objective-c
- (void)testNeedToShowRowWhenTypeIsAll{
    //given
    _sut.collectionType = 1;
    NSIndexPath *path1 = [NSIndexPath indexPathForRow:1 inSection:1];
    
    //when
    BOOL needToShow1 = [_sut needToShowRow:path1];
    
    //then
    assertThatBool(needToShow1,isTrue());
}
```
这样我们一眼扫过去就可以清晰的看出一个case大体上都在干什么。

##2.一个Case只测试一种情况
可能我们调用的一个API内部有一个if...else...。我建议if一个case，else一个case。分两个不同的case来作测试.这样每个case就很清晰自己在测试什么东西。而如果全部杂糅在一个case中，可读性会降低不少，而且case体积也会变得相对大很多，因为你要Given-When-Then两次。更不建议在case中写for循环验证。有人说我的测试目标函数中有很多if...else...,那么我觉得你应该重构下你的设计了。

所以，我们的结论是一个Case只测试一种情况，不同情况用When标明：

```objective-c
- (void)testNeedToShowRowWhenTypeIsAll{
   ...
}

- (void)testNeedToShowRowWhenTypeIsOnSell{
   ...
}
```

##3.用_sut来标明被测试类
一个测试文件只有一个被测试类。但是当我们的测试文件越来越多的时候，当我们看一个测试case的时候需要看懂这个case才明白我们的被测试类是谁。或者我们也可以看测试文件名（XXXXXXTest.m）才知道我们的被测试类是谁，但是这样却不是很直观。所以不管我们在那个测试文件中，测试的类是谁，叫什么名字，我们都以为一个局部变量名_sut来定义我们的被测试类。这样我们一眼就能知道我们被测试类是谁。

_sut就是System Under Test的缩写。

```objective-c
@implementation JHSCollectionDataSourceTest {
    JHSCollectionDataSource *_sut;
}

- (void)testNeedToShowHeaderWhenTypeIsAll{
    //given
    _sut.collectionType = 1;
    
    //when
    BOOL needToShow1 = [_sut needToShowHeader:1];
    
    //then
    assertThatBool(needToShow1,isTrue());
}
```

##4.用Category暴露私有函数和属性
我们的测试case中调用的方法可能会改变一个私有的属性，调用一个私有的方法。怎么去优雅的验证这种行为呢，我们可以在测试文件的开头用一个名字为UnitTest的category来暴露出我们的私有方法和属性（属性暴露的是属性对应的getter和setter方法）。
```objective-c
@interface JHSTestDataSource (UnitTest)
- (NSInteger)getSellGroupCount;
- (BOOL)needShowHeader:(NSInteger)section;
@end
```

##总结
**enjoy it！**




