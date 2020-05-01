# 这里记录我学习Flutter一些坑



# 状态管理Provider

Provider 从名字上就很容易理解，它就是用于提供数据，无论是在**<font color=Red>单个页面</font>**还是在**<font color=Red>整个 app </font>**都有它自己的解决方案，我们可以很方便的管理状态。可以说，Provider 的目标就是**完全替代** StatefulWidget。

我们来看一下 [官方示例](https://pub.flutter-io.cn/packages/provider#-readme-tab-)

## Provider分类
你也可以在 main 方法中通过下面这行代码来禁用此提示。 Provider.debugCheckInvalidValueType = null;

这是由于 Provider 只能提供恒定的数据，不能通知依赖它的子部件刷新。提示也说的很清楚了，假如你想使用一个会发生 change 的 Provider，请使用下面的 Provider。

. ListenableProvider
. ChangeNotifierProvider
. ValueListenableProvider
. StreamProvider
这几个 Provider 有什么异同。

### ListenableProvider / ChangeNotifierProvider 
ListenableProvider 提供（provide）的对象是继承了 Listenable 抽象类的子类。由于无法混入，所以通过继承来获得 Listenable 的能力，同时**<font color=blue>必须</font>**实现其 addListener / removeListener 方法，手动管理收听者。显然，这样太过复杂，我们通常都不需要这样做。

class ChangeNotifier implements Listenable

**而混入了 ChangeNotifier 的类自动帮我们实现了听众管理，所以 ListenableProvider 同样也可以接收混入了 ChangeNotifier 的类。**

ChangeNotifierProvider 则更为简单，它能够对子节点提供一个 继承 / 混入 / 实现 了 ChangeNotifier 的类。通常我们只需要在 Model 中 with ChangeNotifier ，然后在需要刷新状态的时候调用 notifyListeners 即可。

![image](https://gitee.com/mizi23/FlutterNote/raw/master/images/1.png)


那么 ChangeNotifierProvider 和 ListenableProvider 究竟**区别在哪呢**，ChangeNotifierProvider 会在你需要的时候，自动调用其 _disposer 方法。

static void _disposer(BuildContext context, ChangeNotifier notifier) => notifier?.dispose();

我们可以在 Model 中重写 ChangeNotifier 的 dispose 方法，来释放其资源。

### ValueListenableProvider。
ValueListenableProvider 用于提供实现了 继承 / 混入 / 实现 了 ValueListenable 的 Model。它实际上是专门用于处理只有一个单一变化数据的 ChangeNotifier。

class ValueNotifier<T> extends ChangeNotifier implements ValueListenable<T>

**通过 ValueListenable 处理的类<font color=blue>不再需要</font>数据更新的时候调用 notifyListeners。**

### StreamProvider

StreamProvider 专门用作提供（provide）一条 Single Stream。我在这里仅对其核心属性进行讲解。

- T initialData：你可以通过这个属性声明这条流的初始值。

- ErrorBuilder<T> catchError：这个属性用来捕获流中的 error。在这条流 addError 了之后，你会能够通过 T Function(BuildContext context, Object error) 回调来处理这个异常数据。实际开发中它非常有用。

- updateShouldNotify：和之前的回调一样，这里不再赘述。

  

  除了这三个构造方法都有的属性以外，<font color=blue>StreamProvider 还有三种不同的构造方法</font>。

- StreamProvider(...)：默认构造方法用作创建一个 Stream 并收听它。

- StreamProvider.controller(...)：通过 builder 方式创建一个 StreamController<T>。并且在 StreamProvider 被移除时，自动释放 StreamController。

- StreamProvider.value(...)：监听一个已有的 Stream 并将其 value 提供给子孙节点。

## 优雅地处理多个 Provider ---- MultiProvider

![image](https://gitee.com/mizi23/FlutterNote/raw/master/images/2.png)


## Provider 是如何做到状态共享的
  这个问题实际上得分两步。

  ### 获取顶层数据
  实际上在祖先节点中共享数据这件事我们已经在之前的文章中接触过很多次了，都是通过系统的 InheritedWidget 进行实现的。Provider 也不例外，在所有 Provider 的 build 方法中，返回了一个 InheritedProvider。

  class InheritedProvider<T> extends InheritedWidget

  Flutter 通过在每个 Element 上维护一个 InheritedWidget 哈希表来向下传递 Element 树中的信息。通常情况下，多个 Element 引用相同的哈希表，并且该表仅在 Element 引入新的 InheritedWidget 时改变。时间复杂度为 O(1) 。

  ### 通知刷新
  通知刷新这一步实际上就是使用了 Listener 模式。Model 中维护了一堆听众，然后 notifiedListener 通知刷新。

  **<font color=red>全局状态需要放在顶层 MaterialApp 之上，优先初始化，以便在 Navigator 以及 BuildContex控制全局状态</font>**

# State
在Flutter中一切皆组件，也就是Flutter中的Widget。组件又大致可以被分为两类：statelessWidget和statefulWidget。<font color=red>其中有状态的Widget StatefulWidget和StatelessWidget最大的区别就在于StatefulWidget可以通过setState()改变数据使页面动态的更新。</font>

StatefulWidget通过State保存在生命周期中可能发生变化的数据集,StatefulWidget 的 State 帮我们实现了在 Widget 的跨帧绘制 ，也就是在每次 Widget 重绘的时候，通过 State 重新赋予 Widget 需要的绘制信息。

![image](https://github.com/TomMi23/flutternote/blob/master/images/state/1.png)

![image](https://github.com/TomMi23/flutternote/blob/master/images/state/2.png)

statefulWidget通过使用createElement（）创建一个StatefulElement来管理statefulWidget，在statefulElement中保存state。

![image](https://github.com/TomMi23/flutternote/blob/master/images/state/3.png)

当我们调用setState()时就会触发StateElement的update（）将改变后的新Widget重新赋给state的_widget之后在下一帧 WidgetsBinding.drawFrame 重新绘制，达到更新界面的效果。

###  参考资料
[Flutter中的State和状态管理框架Provider学习记录](https://blog.csdn.net/qq_42848018/article/details/103872851?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-34&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-34)

[Flutter状态管理之provide和provider的使用区别](https://www.jianshu.com/p/bea8e9c57666)



# StatefulWidget 生命周期

~~~ dart
import 'package:flutter/material.dart';

class CouterWidgetState extends StatefulWidget {
  const CouterWidgetState({Key key, @required this.initValue: 0})
      : super(key: key);
  final int initValue;
  @override
  State<StatefulWidget> createState() => new CouterState();
}

/// 创建并打开：initState->didChangeDependencies->build.   
/// 横竖屏切换：didUpdateWidget->build  当前值保留
/// 离开页面：deactivate->dispose 重新进入init重新初始化
/// 热重载执行：reassemble->didUpdateWidget->build
/// 调用setState->build
class CouterState extends State<CouterWidgetState> {
  int _couter;
  /// 初始化调用
  @override
  void initState() {
    super.initState();
    _couter = widget.initValue;
    print('initState$_couter');

  }

  /// State对象依赖发生变化调用；系统语言、主题修改，系统也会通知调用
  @override
  void didChangeDependencies() {
    // TODO: implement didChangeDependencies
    super.didChangeDependencies();
    print('didChangeDependencies');
  }
  /// 热重载会被调用，在release下永远不会被调用
  @override
  void reassemble() {
    // TODO: implement reassemble
    super.reassemble();
    print('reassemble');
  }

  /// 新旧Widget的key、runtimeType不变时调用。也就是Widget.canUpdate=>true
  @override
  void didUpdateWidget(CouterWidgetState oldWidget) {
    // TODO: implement didUpdateWidget
    super.didUpdateWidget(oldWidget);
    print('didUpdateWidget');
  }

  /// State在树种移除调用
  @override
  void deactivate() {
    // TODO: implement deactivate
    super.deactivate();
    print('deactivate');
  }

  /// State在树中永久移除调用，相当于释放
  @override
  void dispose() {
    // TODO: implement dispose
    super.dispose();
    print('dispose');
  }

  /// 用于子树的渲染
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    print('build');
    return Scaffold(
      body: Center(
        child: FlatButton(
          child: Text('$_couter'),
          onPressed: (){
            setState(() {
             ++_couter; 
            });
          },
        ),
      ),
    );
  }
}
~~~


![image](https://github.com/TomMi23/flutternote/blob/master/images/StatefulWidget/1.png)



# 获取SHA1方法

https://blog.csdn.net/w13576267399/article/details/83007537

开发版SHA1.

在Terminal 中输入 keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android 然后就能看到下图结果，从中获取到开发版的SHA1值 （如果提示要输入密码，那就是默认密码 android）



 获取正式版的SHA1

还是在 Terminal 中 ，输入 **keytool -list -v -keystore** 后面加个空格 再跟上你打正式包后的 jks 文件完整地址，文件地址获取如下图（文件右键后点击显示简介 进入下图）

#  WidgetsFlutterBinding.ensureInitialized()干什么了？

## WidgetsFlutterBinding是什么？

WidgetsFlutterBinding是什么，从这个类的名称来看，是把Widget和Flutter绑定在一起的意思。
~~~ dart
class WidgetsFlutterBinding extends BindingBase with GestureBinding, ServicesBinding, SchedulerBinding, PaintingBinding, SemanticsBinding, RendererBinding, WidgetsBinding {
  static WidgetsBinding ensureInitialized() {
    if (WidgetsBinding.instance == null)
      WidgetsFlutterBinding();
    return WidgetsBinding.instance;
  }
}
~~~

这个类继承自BindingBase并且混入（Mixin）了很多其他类，看名称都是不同功能的绑定。而静态函数ensureInitialized()所做的就是返回一个WidgetsBinding.instance单例。

混入的那些各种绑定类也都是继承自抽象类BindingBase的

~~~ dart
abstract class BindingBase {
    BindingBase() {
        ...
        initInstances();
        ...
    }
    ...
    ui.Window get window => ui.window;
}
~~~
关于抽象类BindingBase，你需要了解两个地方，一个是在其构造的时候会调用函数initInstances()。这个函数会由其子类，也就是上面说那些各种混入（Mixin）的绑定类各自实现，具体的初始化都是在其内部实现的。另一个就是BindingBase有一个getter，返回的是window。

这些个绑定其实就是对window的封装？来，让我们挨个看一下这几个绑定类在调用initInstances()的时候做了什么的吧。

- 第一个是GestureBinding。手势绑定。
~~~ dart
mixin GestureBinding on BindingBase implements HitTestable, HitTestDispatcher, HitTestTarget {
  @override
  void initInstances() {
    super.initInstances();
    _instance = this;
    window.onPointerDataPacket = _handlePointerDataPacket;
  }
~~~
在调用initInstances()的时候，主要做的事情就是给window设置了一个手势处理的回调函数。所以这个绑定主要是负责管理手势事件的。

- 第二个是ServicesBinding。服务绑定
~~~ dart
mixin ServicesBinding on BindingBase {
 @override
 void initInstances() {
   super.initInstances();
   _instance = this;
   window
     ..onPlatformMessage = BinaryMessages.handlePlatformMessage;
   initLicenses();
 }
~~~
这个绑定主要是给window设置了处理Platform Message的回调。

- 第三个是SchedulerBinding。调度绑定。
~~~ dart
mixin SchedulerBinding on BindingBase, ServicesBinding {
@override
void initInstances() {
  super.initInstances();
  _instance = this;
  window.onBeginFrame = _handleBeginFrame;
  window.onDrawFrame = _handleDrawFrame;
  SystemChannels.lifecycle.setMessageHandler(_handleLifecycleMessage);
}
~~~ 
这个绑定主要是给window设置了onBeginFrame和onDrawFrame的回调，回忆一下上一篇文章讲渲染流水线的时候，当Vsync信号到来的时候engine会回调Flutter的来启动渲染流程，这两个回调就是在SchedulerBinding管理的。

- 第四个是PaintingBinding。绘制绑定。
~~~ dart
mixin PaintingBinding on BindingBase, ServicesBinding {
@override
void initInstances() {
  super.initInstances();
  _instance = this;
  _imageCache = createImageCache();
}
~~~ 
这个绑定只是创建了个图片缓存，就不细说了。

- 第五个是SemanticsBinding。辅助功能绑定。
~~~ dart
mixin SemanticsBinding on BindingBase {
@override
void initInstances() {
  super.initInstances();
  _instance = this;
  _accessibilityFeatures = window.accessibilityFeatures;
}
~~~

这个绑定管理辅助功能，就不细说了。

- 第六个是RendererBinding。渲染绑定。这是比较重要的一个类。
~~~ dart 
mixin RendererBinding on BindingBase, ServicesBinding, SchedulerBinding, GestureBinding, SemanticsBinding, HitTestable {
 @override
 void initInstances() {
   super.initInstances();
   _instance = this;
   _pipelineOwner = PipelineOwner(
     onNeedVisualUpdate: ensureVisualUpdate,
     onSemanticsOwnerCreated: _handleSemanticsOwnerCreated,
     onSemanticsOwnerDisposed: _handleSemanticsOwnerDisposed,
   );
   window
     ..onMetricsChanged = handleMetricsChanged
     ..onTextScaleFactorChanged = handleTextScaleFactorChanged
     ..onPlatformBrightnessChanged = handlePlatformBrightnessChanged
     ..onSemanticsEnabledChanged = _handleSemanticsEnabledChanged
     ..onSemanticsAction = _handleSemanticsAction;
   initRenderView();
   _handleSemanticsEnabledChanged();
   assert(renderView != null);
   addPersistentFrameCallback(_handlePersistentFrameCallback);
   _mouseTracker = _createMouseTracker();
 }
~~~
这个绑定是负责管理渲染流程的，初始化的时候做的事情也比较多。
首先是实例化了一个PipelineOwner类。这个类负责管理驱动我们之前说的渲染流水线。随后给window设置了一系列回调函数，处理屏幕尺寸变化，亮度变化等。接着调用initRenderView()。

~~~ dart
  void initRenderView() {
   assert(renderView == null);
   renderView = RenderView(configuration: createViewConfiguration(), window: window);
   renderView.scheduleInitialFrame();
 }
~~~
这个函数实例化了一个RenderView类。RenderView继承自RenderObject。我们都知道Flutter框架中存在这一个渲染树（render tree）。这个RenderView就是渲染树（render tree）的根节点，这一点可以通过打开"Flutter Inspector"看到，在"Render Tree"这个Tab下，最根部的红框里就是这个RenderView。

最后调用addPersistentFrameCallback添加了一个回调函数。请大家记住这个回调，渲染流水线的主要阶段都会在这个回调里启动。

- 第七个是WidgetsBinding，组件绑定。
~~~ dart
mixin WidgetsBinding on BindingBase, SchedulerBinding, GestureBinding, RendererBinding, SemanticsBinding {
  @override
  void initInstances() {
    super.initInstances();
    _instance = this;
    buildOwner.onBuildScheduled = _handleBuildScheduled;
    window.onLocaleChanged = handleLocaleChanged;
    window.onAccessibilityFeaturesChanged = handleAccessibilityFeaturesChanged;
    SystemChannels.navigation.setMethodCallHandler(_handleNavigationInvocation);
    SystemChannels.system.setMessageHandler(_handleSystemMessage);
  }
~~~

这个绑定的初始化先给buildOwner设置了个onBuildScheduled回调，还记得渲染绑定里初始化的时候实例化了一个PipelineOwner吗？这个BuildOwner是在组件绑定里实例化的。它主要负责管理Widget的重建，记住这两个"owner"。他们将会Flutter框架里的核心类。接着给window设置了两个回调，因为和渲染关系不大，就不细说了。最后设置SystemChannels.navigation和SystemChannels.system的消息处理函数。这两个回调一个是专门处理路由的，另一个是处理一些系统事件，比如剪贴板，震动反馈，系统音效等等。
至此，WidgetsFlutterBinding.ensureInitialized()就跑完了，总体上来讲是把window提供的API分别封装到不同的Binding里。我们需要重点关注的是SchedulerBinding，RendererBinding和WidgetsBinding。这3个是渲染流水线的重要存在。

https://juejin.im/post/5c7e3e70e51d4541c5373143 参看这文章



