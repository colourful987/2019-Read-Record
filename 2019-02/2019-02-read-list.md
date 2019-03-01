#  ![](https://flutter.dev/assets/flutter-lockup-4cb0ee072ab312e59784d9fbf4fb7ad42688a7fdaea1270ccf6bbf4f34b7e03f.svg) Theme 

# 2019/02/01 - 2019/02/13

断更了有段时间，简单罗列下近期做的事，看的书。

* 买了块白板，主要用来罗列知识点和手撸算法，直观、容易加强记忆力；
* 阅读了《人月神话》、《终极算法》以及《码农翻身》三本书，其中《码农翻身》算是最大的收获；另外该书中推荐的《陈寅恪与傅斯年》和《潜规则 中国历史中的真实游戏》这两天也快到了，看看对不对口；
* HTTP，TCP/IP 知识依旧是盲区，因此重新阅读了阮一峰的互联网入门一、二系列文章，这里再次推荐一波阮神的文章，通俗易懂，入门佳作！但是此次二读时发现收获甚少，原因如下：文章本身就极大简化了互联网七层（五层）协议模型，基本是挑重点在讲述，非常适合newbies，但是很多细节都没有讲到，比如三次握手、四次挥手，TCP层传包、拆包等都没有讲到，PS：三次握手的作用在码农一书中真的是一语中的，就是确认双方的send和receive能力，之前看过的很多文章都在讲理论，而且基本看过就忘；
* 又要开始搞模块化/组件化东西了，所以拾下 CTMeditor 等几个方案；
* 分时K线模块化，nmb;

> 之后可能会陆陆续续吧，今年重点放在实战写东西上。

#  - 2019/02/25

[Learning Flutter](https://flutter.dev/docs/get-started/learn-more) ，环境部署、IDE选择、入门教程学习三板斧。

1. 环境部署上遇到点问题，我的brew貌似出了点问题，报错如下，但是`flutter` 命令倒是可以正常执行，所以暂时忽略了。

   ```shell
   ✗ ideviceinstaller is not installed; this is used to discover connected iOS devices.
   To install with Brew, run:
   brew install --HEAD usbmuxd
   brew link usbmuxd
   brew install --HEAD libimobiledevice
   brew install ideviceinstaller
   ✗ ios-deploy not installed. To install with Brew:
   brew install ios-deploy
   ```

2. IDE 选择了 [Visual Studio Code](https://github.com/Microsoft/vscode)， 微软开源，不妨一看，我感觉这个IDE 相比 Sublime Text 大同小异，许多操作都很相似，但vscode安装环境似乎更方便一些；
3. 入门教程，目前就看了官方的[demo](https://flutter.dev/docs/get-started/learn-more)， 分为part1和part2，前者在flutter官方，后者在google的codelab上，学习到的知识点：一、Flutter 语法和基础组件使用，核心是Widget+State ；二、Dart语法；三、package manager，有点类似 cocoapods 的Podfile

> flutter 初识总结：没有语法糖，写Demo的时候疯狂嵌套，这是一种折磨，当然你可以extract local variable，然后再把多个widget作为参数组成一个新的widget，而这个新的widget作为另外一个部件的参数传入。。。个人认为就是组合，可变可配的都抽离出去了，然后随意组。

接下去继续入门教程学习，官方提供的 Next steps:

- [Building Layouts in Flutter](https://flutter.io/tutorials/layout/) tutorial
- [Add Interactivity](https://flutter.io/tutorials/interactive/) tutorial
- [A Tour of the Flutter Widget Framework](https://flutter.io/widgets-intro/)
- [Flutter for Android Developers](https://flutter.io/flutter-for-android/)
- [Flutter for React Native Developers](https://flutter.io/flutter-for-react-native/)
- [Flutter for Web Developers](https://flutter.io/web-analogs/)
- [Build Native Mobile Apps with Flutter](https://www.udacity.com/course/build-native-mobile-apps-with-flutter--ud905) (a free Udacity course)
- [Flutter Cookbook](https://flutter.io/cookbook/)
- [From Java to Dart](https://codelabs.developers.google.com/codelabs/from-java-to-dart/#0) codelab
- [Bootstrap into Dart: learn more about the language](https://flutter.io/bootstrap-into-dart/)

# 2019/02/26

[官方教程Learning...](https://flutter.dev/docs/development/ui/layout/tutorial)

* [x] [Introduction to Widgets](https://flutter.dev/docs/development/ui/widgets-intro)
* [x] [Building layouts](https://flutter.dev/docs/development/ui/layout)
* [x] [Adding interactivity](https://flutter.dev/docs/development/ui/interactive)
* [x] [Assets and images](https://flutter.dev/docs/development/ui/assets-and-images)
* [x] [Navigation & routing](https://flutter.dev/docs/development/ui/navigation) 
* [x] [Return data from a screen](https://flutter.dev/docs/cookbook/navigation/returning-data)
* [ ] [Animations](https://flutter.dev/docs/development/ui/animations)(略过暂时)
* [x] [Advanced UI](https://flutter.dev/docs/development/ui/advanced)
* [x] [Data & backend](https://flutter.dev/docs/development/data-and-backend)
* [x] [Packages & plugins](https://flutter.dev/docs/development/packages-and-plugins)
* [ ] [Tools & techniques](https://flutter.dev/docs/development/tools)

预计这个星期看完基础教程，之后看下Flutter源码实现，这个Theme也算完结了。

# 2019/02/27

学习flutter的第三天，Widget的声明定义基本熟悉，但是一直差异为啥WidgetState定义的最后要 `extends WidgetStateName<WidgetName>`，解释器会动态绑定State到Widget上？State总是维护Widget的状态，所以维护了一些Variable变量，但是State的build总是生成了对应的Widget，所以视图这块还是放在了State中，而Widget往往总是一个空的定义——当然某些状态由ParentWidget管理时，会将状态变量放到Widget中。

> 总的来说，对于Flutter Widget State 分离这块设计还是一知半解，或者说为何这么设计，这么设计的优点在哪里并不是很清楚。



# 2019/02/28

两个页面传值学习，Animation 章节略过，应该也是用Foundation库来实现，另外搜了下关于flutter的曲线库，找到一个[google charts](https://github.com/google/charts/blob/master/charts_flutter/lib/flutter.dart) 和[mzimmerm/**flutter_charts**](https://github.com/mzimmerm/flutter_charts)



# 2019/03/01

> “一星期学上手 Flutter 实战”。😂😂😂

玩笑归玩笑，五天按部就班的部署环境，看官方 tutorial 和接口码代码，所有demo我都放在了github上，目前都是官方给的教程demo。对于一些章节，如[Animations](https://flutter.dev/docs/development/ui/animations) 、[Tools & techniques](https://flutter.dev/docs/development/tools)、发布自己Package的流程我都选择了暂时略过，有需再取。

下面谈谈对flutter上手体验：

1. 环境部署：难度⭐️，vscode一键安装，或者Github直接下载源码，然后配下环境PATH搞定，但是在运行 `flutter docter` 命令时估计大部分会遇到问题，不过flutter给出的帮助信息还不错，缺少依赖什么的都会告诉你，解决步骤也附上用 `brew install xx...`，**目前只部署了iOS环境，安卓无需就暂时不搞了，之后补；**

2. Dart 语言，此次学习并没有按部就班，先学完基础语法 《Dart Programming Language Tour》后再开始做App，主要是觉得语言这块都是相通的，一些特殊语法在使用过程中可能没接触过，此时会查询下官方文档；

3. FLutter 应用程序上手，感觉嵌套到让人怀疑人生！组合，组合，组合！即使IDE提供了format以及注解region，还是觉得治标不治本；

4. 编程习惯： imperative programming 和 declarative programming的区别，flutter作为后者核心可以用下图表示，state和widget，每一次state变更，都会根据状态重新渲染生成新状态下的widget，核心方法就是我们每次override的`widget build(BuildContext context){}`方法

   

   ![](https://flutter.dev/assets/development/data-and-backend/state-mgmt/ui-equals-function-of-state-54b01b000694caf9da439bd3f774ef22b00e92a62d3b2ade4f2e95c8555b8ca7.png)

5. package manager，这点感觉和cocoapods以及swift package manager很像；

6. 内存以及渲染：内存应该是会大一些的，毕竟内存存储了所有状态和widget信息，上次看到文章说还不释放Route(等同于viewController)，渲染的话目前感受不出，大家都说不错，原因是渲染那一套google自己实现的，orz…

接下来是源码阅读，简单猜测下：

1. Flutter SDK 内置解释器；
2. 存储了flutter所有widget和state信息，可能还维护了几个视图树，有点类似oc中 `presentLayer *Tree*`、`modeLayer Tree`和`Render Tree`；
3. 渲染系统基于 Quartz；





