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

