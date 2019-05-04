# 2019/05

> 忙碌的五月



### 模块化/组件化

两者的定义先按下不表，目前大部分模块化/组件化方案想要解决的问题：往"糙"了说就是在不引入依赖的前提下（不需要写`#import`）得到一个`ViewController` 实例；当然解耦是关键问题，其次还有一些方案中的 Mediator 承担的职责会更多一些，甚至视图控制器的跳转/Push跳转都需要它来协调，当然此处不可避免的是某些跳转之前或者之后需要一些业务处理，如果写到Mediator单例中就违背了解耦——引入了业务，所以通常中间者会切片告知业务方一些关键Event，业务只需要在程序一开始就注册进来(毫无以为是用协议，一般会在`+load()`方法)；CTMediator 一开始有个问题就是如果用Runtime反射确实可以做到解耦，但是调用处必须使用硬编码的字符串感觉不够友好，所以会引用协议注入方式，这样做又不可避免引入依赖——必须引入一个定义了协议的头文件，今天要讨论的就是这种方式：

```objective-c
static NSMutableDictionary *dic = nil;

NS_INLINE id InstanceFromProtocol(Protocol *protocol){
		NSString *className = NSStringFromProtocol(protocol);
    Class aClass = NSClassFromString(className);
    
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        dic = [NSMutableDictionary dictionary];
    });
  
    id module = dic[className];
    if (!module) {
        module = [[aClass alloc]init];
        dic[className] = module;
    }
    if ([module conformsToProtocol:protocol]){
        return module;
    }
    return nil;
}
```

如果在引入了协议的依赖文件后，依旧得到的是`id`类型的实例，那么我们还是无法得知这个实例提供了哪些方案——可以理解为还是没有代码补全，想要实现这种效果，很简单，做一个强转操作才可以，当然我们不可能在每个地方都调用 `[(id<XXXProtocol>)idInstance methodDefinedInXXXProtocol]`，因此我们需要一个宏：

```c
#define DefineInstance(varName,ModuleProtocol) id<ModuleProtocol> varName = InstanceFromProtocol(@protocol(ModuleProtocol));

#define GetModuleInstance(ModuleProtocol) ((id<ModuleProtocol>)(InstanceFromProtocol(@protocol(ModuleProtocol))))
```

现在按照上面的思路，完整思路：

```objective-c
@interface YourModuleName<NSObject>
// 你的模块提供的接口方案，或者属性
@end

@implementation YourModuleName
// 接口实现
@end

// ==================================
@protocol YourModule<NSObject>
// 和 `YourModuleName` 一模一样，毕竟最后调用方是看这个协议提供的接口
@end
```

最后说下 CTMediator 以及类Mediator的方案中经常用到的：

```objective-c
@implementation XXModule

- (void)gotoXXViewControllerFromViewController:(UIViewController *)viewController {
    XXViewController *userChatVC = [[XXViewController alloc]init];
    [viewController.navigationController pushViewController:userChatVC animated:YES];
}

@end
```

本文中每个模块都有一个对外打交道的类，这里是`XXModule`，这个类中会自己实现一些业务处理，做一些跳转等等；每次Caller想要调用其他组件的功能，都必须先实例化这么一个 `XXModule` 的实例对象来，由它来执行对应的功能处理。

看到有些文章说"组件协议里的方案都要写成实例方法，不要写成类方法"，存疑下，要说是因为状态存储问题那缺失是不能，其他似乎也没影响吧。。。

































