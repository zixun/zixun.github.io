---
layout: post
title: "ZXOptionBar：模拟UITableView实现Cell重用以及视图数据分离的横向滑动视图"
date: 2014-09-01 13:39:53 +0800
comments: true
categories: 
---

#Overview
ZXOptionBar是模仿twitter开源的[twui](https://github.com/twitter/twui)实现UIKit中UITableView的Cell重用机制以及数据和视图的分离的横向滚动视图控件。</br>
该控件与UITableView的区别：</br>
1.UITableView是纵向滚动，ZXOptionBar是横向滚动。</br>
2.UITableView分很多section，ZXOptionBar没有section。因为横向滚动如果有section的View的话个人感觉不美观，而且一般横向滚动的cell内容都是选项。</br>
3.ZXOptionBar有选中指示器indicator。</br>

**NOTE:**接下去的步骤只是一些关键逻辑的代码，如想看完整代码，请拉倒最下面的Github地址
#Step1：定义设置delegate和DataSource接口

```objective-c
@protocol ZXOptionBarDataSource
@required
-(NSInteger) numberOfColumnsInOptionBar:(ZXOptionBar *)optionBar;
-(ZXOptionBarCell *)optionBar:(ZXOptionBar *)optionBar cellForColumnAtIndex:(NSInteger) index;
@end
```

```objective-c
@protocol ZXOptionBarDelegate <NSObject,UIScrollViewDelegate>
-(CGFloat)optionBar:(ZXOptionBar*)optionBar widthForColumnsAtIndex:(NSInteger) index;
-(void) optionBar:(ZXOptionBar *)optionBar didSelectColumnAtIndex:(NSInteger) index;
-(void) optionBar:(ZXOptionBar *)optionBar didDeselectColumnAtIndex:(NSInteger)index;
-(void) optionBar:(ZXOptionBar *)optionBar willDisplayCell:(ZXOptionBarCell*)cell forColumnAtIndex:(NSInteger) index;
- (void)optionBarWillReloadData:(ZXOptionBar *)optionBar;
- (void)optionBarDidReloadData:(ZXOptionBar *)optionBar;
/*
 *optionBar从一个cell的选中状态转换到另一个cell的选中状态的进度
 *fIndex:初始选中cell index
 *tIndex:最后选中cell index
 *progress:进度
 */
-(void) optionBar:(ZXOptionBar *)optionBar selectFromIndex:(NSInteger) fIndex toIndex:(NSInteger)tIndex progress:(CGFloat) progress;
@end
```

在我们使用UITableView的时候我们的我们会写类似```self.tableView.delegate = self```这样的代码。你应该也发现了我们只设置了我们UITableView的delegate但是self却能回调UIScrollViewDelegate中的方法，我们去看UITableViewDelegate的声明
``` 
@protocol UITableViewDelegate<NSObject, UIScrollViewDelegate>
```
 可以发现UITableViewDelegate继承自UIScrollViewDelegate，很巧妙的把UIScrollViewDelegate抛给了代理者。

我们的ZXOptionBar中也是通过继承UIScrollViewDelegate来实现一样的效果。但是在delegate的get和set中必须调用super相应的方法。

```objective-c
@property(nonatomic,weak) id<ZXOptionBarDelegate> delegate;
@property(nonatomic,weak) id<ZXOptionBarDataSource> dataSource;
```

```objective-c
-(id<ZXOptionBarDelegate>) delegate{
    return (id<ZXOptionBarDelegate>)[super delegate];
}
-(void) setDelegate:(id<ZXOptionBarDelegate>)delegate{
    [super setDelegate:delegate];
}
```


#Step2：重写layoutSubViews方法实现Cell重用

在我们的逻辑中，layoutSubViews方法其中两种情况下会被触发，一种是ZXOptionBar在initWithFrame时（Frame参数不是CGRectZero），另外一种就是UIScrollView滚动的时候，也就是ZXOptionBar滚动的时候。所以我们可以利用重写layoutSubViews来计算和布局我们重用的cell视图。

首先，我们可以创建两个容器变量来存放我们要用到的数据，一个用来存放当前正在显示的可见Cell（_visibleItems），一个用来存放可重用的待显示的Cell（）：
```objective-c
NSMutableDictionary *_reusableOptionCells;
NSMutableDictionary *_visibleItems;
```
并且在初始化方法中初始化它
```objective-c
_reusableOptionCells = [[NSMutableDictionary alloc]init];
NSMutableDictionary *_visibleItems;
```
初始化好后我们可以构造一些操作这两个变量的实例方法用于辅助，作用与方法名一致：
```objective-c
- (NSArray *)indexsForVisibleColumns
{
	return [_visibleItems allKeys];
}
```

```objective-c
-(ZXOptionBarCell*) cellForColumnAtIndex:(NSInteger)index{
    return [_visibleItems objectForKey:[NSNumber numberWithInteger:index]];
}
```

```objective-c
-(void)_enqueueReusableCell:(ZXOptionBarCell *)cell
{
	NSString *identifier = cell.reuseIdentifier;
	if(!identifier)
		return;
	NSMutableArray *array = [_reusableOptionCells objectForKey:identifier];
	if(!array) {
		array = [[NSMutableArray alloc] init];
		[_reusableOptionCells setObject:array forKey:identifier];
	}
	[array addObject:cell];
}
```

```objective-c
-(id)dequeueReusableCellWithIdentifier:(NSString*)identifier{
    if (!identifier) {
        return nil;
    }
    NSMutableArray *array = [_reusableOptionCells objectForKey:identifier];
    if (array) {
        ZXOptionBarCell *c = [array lastObject];
        if (c) {
            [array removeLastObject];
            [c prepareForReuse];
            return c;
        }
    }
    return nil;
}
```

接下来我们就可以重写layoutSubviews了：

```objective-c
-(void)layoutSubviews{
    CGRect visible = [self _visibleRect];
    // Example:
	// old:            0 1 2 3 4 5 6 7
	// new:                2 3 4 5 6 7 8 9
	// to remove:      0 1
	// to add:                         8 9
	
    NSArray *oldVisibleIndex = [self indexsForVisibleColumns];
    NSArray *newVisibleIndex = [self indexsForColumnInRect:visible];
    
    NSMutableArray *indexsToRemove = [oldVisibleIndex mutableCopy];
	[indexsToRemove removeObjectsInArray:newVisibleIndex];
	
	NSMutableArray *indexsToAdd = [newVisibleIndex mutableCopy];
	[indexsToAdd removeObjectsInArray:oldVisibleIndex];
    
    
    //删除离开屏幕的cell
    for (NSNumber* i in indexsToRemove) {
        ZXOptionBarCell* cell = [self cellForColumnAtIndex:i.integerValue];
        [self _enqueueReusableCell:cell];
        [cell removeFromSuperview];
        [_visibleItems removeObjectForKey:i];
    }
    
    //添加新cell
    for (NSNumber* i in indexsToAdd) {
        ZXOptionBarCell* cell = [_dataSource optionBar:self cellForColumnAtIndex:i.integerValue];
        cell.frame = [self rectForColumnAtIndex:i.integerValue];
        cell.layer.zPosition =0;
        [cell setNeedsDisplay];
        [cell prepareForDisplay];
        cell.index = i.integerValue;
        if (i.integerValue == _selectedIndex) {
            [cell setSelected:YES animated:NO];
        }else{
            [cell setSelected:NO animated:NO];
        }
        [cell animationWidthGradient:_gradient];
        if (_optionBarFlags.delegateOptionBarWillDisplayCellForColumnAtIndex) {
            [self.delegate optionBar:self willDisplayCell:cell forColumnAtIndex:i.integerValue];
        }
        [self addSubview:cell];
        [_visibleItems setObject:cell forKey:i];
    }  
}
```
#Step3：实现delegate和DataSource方法触发
可以看到Step2中的添加新cell中已经触发了optionBar: cellForColumnAtIndex:和optionBar: willDisplayCell: forColumnAtIndex:两个代理方法
