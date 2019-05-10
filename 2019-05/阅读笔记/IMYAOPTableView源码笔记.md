# IMYAOPTableView 源码阅读笔记

大致过了一遍源码，作者在开头就说明了该库的使用场景：

> 关于为何使用AOP，在[MeetYouDevs/IMYAOPTableView](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2FMeetYouDevs%2FIMYAOPTableView)这个库的简介中已经有提及到了，主要是针对在我们数据流中接入广告的这种场景，最原始的方法就是分别请求数据以及广告，根据规则合并数据，分别处理业务数据和广告数据的展示这个流程如下图所示。这种方案的弊端就是有很明显的耦合，广告和正常的业务耦合在一起了，同时也违反了设计原则中的单一职责原则，所以这种方式是做的不够优雅的，后期的维护成本也是比较大的。

![](https://user-gold-cdn.xitu.io/2019/4/25/16a53eabf6099a01?imageView2/0/w/1280/h/960/ignore-error/1)

如果使用该库：

![](https://user-gold-cdn.xitu.io/2019/4/25/16a53eabf61305bb?imageView2/0/w/1280/h/960/ignore-error/1)



两种方案，各有利弊。第一种方案：分开请求Feed流数据和广告数据—>数据合并—>界面展示，关于作者的耦合不是很清楚，但是如果协议规范，或者使用多态似乎也是可以实现，甚至支持的类型不仅限于Feed流和广告；如果作者纠结的是广告和Feed流请求接口走了两个导致觉得不符合单一原则，是否可以后台统一接口呢？接口返回的就是前段要展示的数据。以上只是我粗浅的想法。

继续回过头说第二种方案，我简单用了下，确实针对两种数据接口的场景用起来很顺手，但是貌似不支持3种以上的数据接口，应该是当初只是为了解决Feed流和广告两种，所以算不上完全支持AOP切面。



## 方案的亮点

Feed流展示信息一般有两种：tableview 和 collectionView，因此作者也只是对这两者做了实现。因为tableview要支持展示feed流和广告信息，所以将处理这两者的逻辑都放到了 `IMYAOPBaseUtils` 中，tableview和collectionview的不同，所以子类化了`IMYAOPTableViewUtils` 和 `IMYAOPCollectionViewUtils` 两种。然后每个 tableview 实例都会绑定一个 `IMYAOPTableViewUtils` 实例，这里借助了`objc_getAssociatedObject` 和 `objc_getAssociatedObject` 。

第一次绑定的时候还需要做一些处理，处理放在了`IMYAOPTableViewUtils`的  `injectTableView` 方法中：

```objective-c
// IMYAOPTableViewUtils.m 文件
- (void)injectTableView {
    UITableView *tableView = self.tableView;

    _origDataSource = tableView.dataSource;
    _origDelegate = tableView.delegate;

    [self injectFeedsView:tableView];
}
```

这里先保存了 tableview 的original datasource和delegate，而在 `injectFeedsView` 中会将 tableview 的delegate和DataSource重新设置为`IMYAOPTableViewUtils` 实例。见下：

```objc
- (void)injectFeedsView:(UIView *)feedsView {
    struct objc_super objcSuper = {.super_class = [self msgSendSuperClass], .receiver = feedsView};
    ((void (*)(void *, SEL, id))(void *)objc_msgSendSuper)(&objcSuper, @selector(setDelegate:), self);
    ((void (*)(void *, SEL, id))(void *)objc_msgSendSuper)(&objcSuper, @selector(setDataSource:), self);

    self.origViewClass = [feedsView class];
    Class aopClass = [self makeSubclassWithClass:self.origViewClass];
    if (![self.origViewClass isSubclassOfClass:aopClass]) {
        [self bindingFeedsView:feedsView aopClass:aopClass];
    }
}
```

考虑到Feed流和广告模块中都会操作 tableview 的方法，这里选择的做法是subclass 原 tableview，也就是这里的 `makeSubclassWithClass`，这个类对象是动态创建，以`kIMYAOP_` 前缀+当前Tableview的类名(这里是YYTableView)，然后将所有Tableview的方法Add到这个新Class类对象中：

```objc
- (void)setupAopClass:(Class)aopClass {
    ///纯手动敲打
    [self addOverriteMethod:@selector(class) aopClass:aopClass];
    [self addOverriteMethod:@selector(setDelegate:) aopClass:aopClass];
    [self addOverriteMethod:@selector(setDataSource:) aopClass:aopClass];
    [self addOverriteMethod:@selector(delegate) aopClass:aopClass];
    [self addOverriteMethod:@selector(dataSource) aopClass:aopClass];
    [self addOverriteMethod:@selector(allowsSelection) aopClass:aopClass];
    
    ///UI Calling
    [self addOverriteMethod:@selector(reloadData) aopClass:aopClass];
    [self addOverriteMethod:@selector(layoutSubviews) aopClass:aopClass];
    [self addOverriteMethod:@selector(setBounds:) aopClass:aopClass];
    [self addOverriteMethod:@selector(bringSubviewToFront:) aopClass:aopClass];
  ...
}
```

往aopClass这个动态创建的类对象中添加方法同样有意思，这里的method——具体的实现都是从 `[self implAopViewClass]` 这个类对象中获取的，以tableview为例就是 `_IMYAOPTableView` 这个类。

```objc
- (void)addOverriteMethod:(SEL)seletor aopClass:(Class)aopClass {
    NSString *seletorString = NSStringFromSelector(seletor);
    NSString *aopSeletorString = [NSString stringWithFormat:@"aop_%@", seletorString];
    SEL aopMethod = NSSelectorFromString(aopSeletorString);
    [self addOverriteMethod:seletor toMethod:aopMethod aopClass:aopClass];
}

- (void)addOverriteMethod:(SEL)seletor toMethod:(SEL)toSeletor aopClass:(Class)aopClass {
    Class implClass = [self implAopViewClass];
    Method method = class_getInstanceMethod(implClass, toSeletor);
    if (method == NULL) {
        method = class_getInstanceMethod(implClass, seletor);
    }
    const char *types = method_getTypeEncoding(method);
    IMP imp = method_getImplementation(method);
    class_addMethod(aopClass, seletor, imp, types);
}
```

来看下 `_IMYAOPTableView` 这个类重写的方法：

```objc
@implementation _IMYAOPTableView

- (void)aop_setDelegate:(id<UITableViewDelegate>)delegate {
    IMYAOPTableViewUtils *aop_utils = self.aop_utils;
    if (aop_utils) {
        if (aop_utils.origDelegate != delegate) {
            AopDefineObjcSuper;
            AopCallSuper_1(@selector(setDelegate:), delegate);
            aop_utils.origDelegate = delegate;
        }
    } else {
        [super setDelegate:delegate];
    }
}

- (BOOL)aop_allowsSelection {
    AopDefineVars;
    aop_utils.isUICalling += 1;
    BOOL allowsSelection = ((BOOL(*)(void *, SEL))(void *)objc_msgSendSuper)(&objcSuper, @selector(allowsSelection));
    aop_utils.isUICalling -= 1;
    IMYAOPGlobalUICalling = YES;
    return allowsSelection;
}

- (id<UITableViewDelegate>)aop_delegate {
    if (IMYAOPGlobalUICalling && self.aop_utils.isUICalling == 0) {
        AopDefineVars;
        return AopCallSuperResult(@selector(delegate));
    }
    AopDefineVars;
    if (aop_utils) {
        return aop_utils.origDelegate;
    } else {
        return AopCallSuperResult(@selector(delegate));
    }
}

- (UITableViewCell *)aop_cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    AopDefineVars;
    if (aop_utils) {
        indexPath = [aop_utils feedsIndexPathByUser:indexPath];
    }
    aop_utils.isUICalling += 1;
    UITableViewCell *cell = AopCallSuperResult_1(@selector(cellForRowAtIndexPath:), indexPath);
    aop_utils.isUICalling -= 1;
    return cell;
}

....
```

所有对原tableview的方法调用，实际上调用的都是 `aop_` 开头的方法，方法内部都会调用 super 的方法——也就是原方法。

关于页面刷新的核心逻辑，前面也说到了tableview的delegate和datasource实际上设置成了`IMYAOPTableViewUtils` 实例：

```
- (void)injectFeedsView:(UIView *)feedsView {
    struct objc_super objcSuper = {.super_class = [self msgSendSuperClass], .receiver = feedsView};
    ((void (*)(void *, SEL, id))(void *)objc_msgSendSuper)(&objcSuper, @selector(setDelegate:), self);
    ((void (*)(void *, SEL, id))(void *)objc_msgSendSuper)(&objcSuper, @selector(setDataSource:), self);
	....
}
```

在 `IMYAOPTableViewUtils+DataSource.h` 中可以找到具体实现：

```objc
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
    kAOPUICallingSaved;
    NSInteger numberOfSection = 1;
    if ([self.origDataSource respondsToSelector:@selector(numberOfSectionsInTableView:)]) {
        numberOfSection = [self.origDataSource numberOfSectionsInTableView:tableView];
    }
    ///初始化回调
    [self.dataSource aopTableUtils:self numberOfSection:numberOfSection];

    ///总number section
    if (numberOfSection > 0) {
        numberOfSection = [self feedsSectionByUser:numberOfSection];
    }
    kAOPUICallingResotre;
    return numberOfSection;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    kAOPUICallingSaved;
    NSInteger userSection = [self userSectionByFeeds:section];
    NSInteger rowCount = 0;
    if (userSection >= 0) {
        section = userSection;
        rowCount = [self.origDataSource tableView:tableView numberOfRowsInSection:section];

        NSIndexPath *feedsIndexPath = [self feedsIndexPathByUser:[NSIndexPath indexPathForRow:rowCount inSection:section]];
        rowCount = feedsIndexPath.row;
    } else {
        NSMutableArray<NSIndexPath *> *array = self.sectionMap[@(section)];
        for (NSIndexPath *obj in array) {
            if (obj.row <= rowCount) {
                rowCount += 1;
            } else {
                break;
            }
        }
    }
    kAOPUICallingResotre;
    return rowCount;
}

// Row display. Implementers should *always* try to reuse cells by setting each cell's reuseIdentifier and querying for available reusable cells with dequeueReusableCellWithIdentifier:
// Cell gets various attributes set automatically based on table (separators) and data source (accessory views, editing controls)

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    kAOPUICallingSaved;
    kAOPUserIndexPathCode;
    UITableViewCell *cell = nil;
    if ([dataSource respondsToSelector:@selector(tableView:cellForRowAtIndexPath:)]) {
        cell = [dataSource tableView:tableView cellForRowAtIndexPath:indexPath];
    }
    if (![cell isKindOfClass:[UITableViewCell class]]) {
        cell = [UITableViewCell new];
        if (dataSource) {
            NSAssert(NO, @"Cell is Nil");
        }
    }
    kAOPUICallingResotre;
    return cell;
}
```

你会发现没找到Feed流和插入广告的分支处理，实际上这些都是在宏里定义了：

```C
#define kAOPUserIndexPathCode                                           \
    NSIndexPath *userIndexPath = [self userIndexPathByFeeds:indexPath]; \
    id<IMYAOPTableViewDataSource> dataSource = nil;                     \
    if (userIndexPath) {                                                \
        dataSource = (id)self.origDataSource;                           \
        indexPath = userIndexPath;                                      \
    } else {                                                            \
        dataSource = self.dataSource;                                   \
        isInjectAction = YES;                                           \
    }                                                                   \
    if (isInjectAction) {                                               \
        self.isUICalling += 1;                                          \
    }

#define kAOPUserSectionCode                                    \
    NSInteger userSection = [self userSectionByFeeds:section]; \
    id<IMYAOPTableViewDataSource> dataSource = nil;            \
    if (userSection >= 0) {                                    \
        dataSource = (id)self.origDataSource;                  \
        section = userSection;                                 \
    } else {                                                   \
        dataSource = self.dataSource;                          \
        isInjectAction = YES;                                  \
    }                                                          \
    if (isInjectAction) {                                      \
        self.isUICalling += 1;                                 \
    }

#define kAOPUICallingSaved          \
    BOOL isInjectAction = NO;       \
    self.isUICalling -= 1;

#define kAOPUICallingResotre        \
    if (isInjectAction) {           \
        self.isUICalling -= 1;      \
    }                               \
    self.isUICalling += 1;
```

今天阅读大致就是这些，还没有细看，方案对于处理两种数据流的tableview还是很适用的，但是还得看项目实际情况而定。也算提供了一种解决问题的方案，很赞。



























