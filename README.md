# 继承、分类、面向协议编程

## 1、 继承

### 什么是「继承」

> 继承是面向对象的四大特性之一，用来表示类之间的 is-a 关系，可以解决代码复用的问题。

简单的例子基本上都在各类书上都有了：

```objective-c
@interface Animal : NSObject
- (void)breath;

@interface Bird : Animal
- (void)fly;

@interface Fish : Animal
- (void)swim;
```

最大的优势在于父类代码的重用，实际项目中大部分的VC继承自```XXXBaseViewController```，其中会包含一些基本的功能，诸如埋点、UI设定、显示消失逻辑处理等等。

### 为什么不推荐使用继承

继承很好的让代码可以复用，但是同样也带来代码高耦合的问题。

举个例子。

从系统类出发，```UIViewController```

一般情况下我们会定制一些默认行为的，```XXXBaseViewController```

然后会有一些通用列表页或者左右滑动页，```XXXTableController```与```XXXSegmentController```

根据不同的使用方式还会出现特定使用模式的子类，```XXXRACTableController```

以上的3次继承中还可以接受，但是一旦进入业务场景

在不同的业务场景会出现不同的业务子类，```XXXIMBaseViewController```、```XXXVideoBaseViewController```

这样情况下大多数的类在经过4-5次继承之后就会造成维护的难度变的相当的高。

看一下这两个VC：

```objective-c
@interface XXIMPersonalChatViewController : XXIMChatViewController
  : XXIMRACTableViewController 
    : XXIMTableViewController
      : XXIMViewController
        : BaseViewController
          : UIViewController
  
@interface XXIMSessionHomeVC : XXIMSessionsViewController
  : XXIMBaseSessionsViewController
    : XXIMRACTableViewController
      : XXIMTableViewController
        : XXIMViewController
          : BaseViewController 
            : UIViewController
```

如此的继承链中，任何一个父类的改动都会造成所有子类的的变化，当然这个也有可能就是你本身的期望。但是如果这并非你的原意，那么这样牵一发而动全身的改动，就是继承高耦合的体现。

比如我们新增加了一种不需要刷新的列表：```XXStaticListViewController```，

```XXIMTableViewController```中已经定义了```-didTriggerRefresh```刷新事件，并在初始化时已经绑定了，

那么我们需要怎么做？





1. 既然```shouldPullToRefresh```可以控制是否需要区分，那么我们的选择可以是在```XXIMTableViewController```这层添加区分属性。
2. 我们可以重写```XXIMTableViewController```中关于刷新的方法。
3. 我们可以从```XXIMTableViewController```这部分开始重新创造一个类来满足需求。



以上方式都会产生大大小小的影响，越来越多的功能会在每一个类上面多多少少的造成影响，导致部分不需要这个功能的子类也会受到影响。

从另一个角度来看，我们需要移除部分功能的时候，就变的更加困难了，如果我的```XXIMPersonalChatViewController```需要更改成另个一业务基类，那么中间两层所持有的功能也需要重新构建。

## 2、分类

### Category的好处

分散不同的实现，可以模拟多继承，不同的功能可以直接组合，避免为了多种组合而造成的继承过多。

``` objective-c
// ViewController+Table.h
@interface UIViewController (Table) <UITableViewDataSource>

@end

// ViewController+Table.m
@implementation UIViewController (Table)

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    return 1;
}

- (nonnull UITableViewCell *)tableView:(nonnull UITableView *)tableView cellForRowAtIndexPath:(nonnull NSIndexPath *)indexPath {
    return [UITableViewCell new];
}

@end

```

通过这样一个分类可以实现通用的TableViewDataSource。但是这样也会有问题。

### Category的局限

只能用runtime添加成员变量。

会覆盖原类方法。

重名方法会被覆盖。

即便没有引用此分类，方法已经被注册进类了，当在发送消息时会执行这个。

## 3、面向协议编程

Swift非常强调面向协议编程的概念。

### 提供干净的重用性

Swift的协议可以通过扩展添加本身的实现，而这个功能可以直接添加，无需通过继承和覆盖。遵循协议的同时也拥有了默认实现，这就很cooool了。

### 实现多继承

协议没有数量限制，可以通过多个协议添加各种通用功能，例如

``` swift
protocol Refreshable { }

protocol LoadMore { }

class XXTableController: Refreshable, LoadMore {
  
}
```

不同的组合方式可以提供不同的功能，在移除时也不用担心耦合。

### 值类型“继承”

结构体类型不能继承，当时可以通过协议完成一些所需的功能。

### 依赖注入

MVVM注入模型是可以使用协议而非子类。



## 4、小结一下

继承作为面向对象的特性已经被使用了很多年了，优势很明显，但是在日益复杂的继承关系下也出现了缺点。主要是在OC，别的语言有多继承会好很多。

Swift的协议很好的解决了这些缺点，我想这个也是Swift是未来趋势的一个原因吧。
