---
title: Android程序员的Flutter学习笔记
date: 2018-11-08 11:17:10
tags: [Android, Flutter]
categories: Flutter
comments: true
description: Android程序员 Flutter学习笔记 Flutter self-learning notes as an android developer
thumbnailImage: https://res.cloudinary.com/dtn0pkdmg/image/upload/v1541647409/flutter_logo_yum7hs.png
---

作为忠实与较资深的Android汪, 最近抽出了一些时间研究了一下Google的亲儿子Flutter, 尚属皮毛, 只能算是个简单的记录吧.

Google自2017年第一次提出Flutter, 到2018年Beta, 再加之RN的各种风波与问题, 使得Flutter的热度不断上升, 国内不少公司都公布Flutter在其产品中的应用, 如美团, 闲鱼等.

<!--more-->

## 前言

Flutter作为跨平台框架, 常常被人拿出来与React Native, 以及Xamarin进行对比, 除了大家都是跨平台框架之外且能达到近乎Native的体验之外, Flutter与这两者的原理大不相同.

让我们来看看这三者的结构图吧.

![React Native](/images/react_native.png)

![Xamarin](/images/xamarin.png)

![Flutter](/images/flutter.png)


可能有一些复杂, 咱大致解释一下.

React Native跟Xamarin都是基于mapping native代码来实现所谓的Native体验的框架, 只是RN基于JS引擎 + Bridge与native打交道, 并且在运行时进行绑定, 而Xamarin是基于微软的基于Linux的C#虚拟机mono + JNI与native进行通信.

这里Android与iOS还是有差别的, 如RN在iOS上JS引擎不支持JIT, 会一定程度影响效率, Xamarin在iOS上可以直接编译成iOS平台可以执行的程序, 所以在实际运行起来的性能是一样的, 唯一的差别就是微软得更快的支持API同步.

对于Flutter来说, 由于他的渲染引擎使用了Skia直绘, 加上基于C++的Dart引擎, 所以在不同平台上没有差别, 加之其实现了Android Material Design与iOS Cupertino两套UI组件, 所以即便是自绘组件, 看起来还是跟原生的一个样子.

通过对三种跨平台引擎的大致了解, 我们可以看出来, 他们都达到了一定程度的Native体验, 然则各自都有一定的性能损耗, 比如RN的JS引擎加载JS, 以及Bridge通信的损耗, Xamarin Mono虚拟机与Java通信的损耗, 以及Flutter Skia渲染与Native Android渲染的差异等.

## Flutter笔记

### 如何启动一个app

Android需要在Manfest里面指定带有MAIN action与LAUNCHER category的Activity声明, 而Flutter只需要一行.

```
void main() => runApp(MyApp());
```

其中MyApp就是一个普通的Widgets(View).

### View vs Widgets

Flutter没有View, 与之对应的是Widget, 并且分为StatelessWidgets与StatefulWidgets, 前者是个静态View, 后者是动态通过Data来更新的View.

- Stateless

```
Text(
  'I like Flutter!',
);
```

- Stateful

```
class StatefulText extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => _TextState();
}

class _TextState extends State<StatefulText> {
  // Default placeholder text
  String textToShow = "I Like Flutter";

  void _updateText() {
    setState(() {
      // update the text
      textToShow = "Flutter is Awesome!";
    });
  }
  @override
  Widget build(BuildContext context) {
      ...invoke _updateText
  }
}
```

实际上是因为StatefulWidgets通过调用`State`的`setState`方法来触发整个Widgets树的重绘, 并且在重绘之前会调用传进去的`(){ ... }`block.


### 怎么写Layout, XML到哪里去了.

实际上Flutter没有xml了, 并且是通过Widgets的嵌套来实现一个布局的.

如:
- `Center`是一个可以把子View放置在中央的容器.
- `Row`对应的就是LinearLayout + Horizontal, `Column`对应的就是LinearLayout + Vertical, 他们都具备一个属性叫做`crossAxisAlignment`, 有点类似`gravity`, 来控制子View相对于父View的位置.
- `Expanded`支持一个类似weight的属性, 叫`flex`. 
- `Container`是一个具有`decoration`属性的容器, 可以用来控制背景色, border, margin等等.
- `Stack`有点像是一个特殊的RelatetiveLayout或者ConstraintLayout, `children`属性指定了它的子View, 第一个是Base View, `alignment`属性指定了后面的子View相对于BaseView的位置, 如`alignment: const Alignment(0.6, 0.6)`指定了位于BaseView右下角的位置.
- `ListTile`是一个特殊的ListItem, 有三个属性, 分别是左边的Icon (leading), 文字 (title), 以及右边的Icon (trailing).
- 还有诸如`ListView`, `GridView`, `Card`等等比较熟悉的Widgets.

另外有一个类似于我们Activity的Widgets:

- 叫做`MaterialApp`, 可以指定`theme`, `title`, 以及子View `home`, 还有更重要的页面跳转`routes`.

```
MaterialApp(
      title: 'Welcome to Flutter',
      home: ...,
      routes: <String, WidgetBuilder> ...,
      theme: ThemeData(
        primaryColor: Colors.white
      ),
    )
```

还有一个类似于Fragment的:

- 叫做`Scaffold`, 中文意思是`脚手架`, 它包含一个appBar (ActionBar)与一个body, appBar可以指定title与actions (类似于action button的点击事件).

```
Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
        actions: <Widget>[...],
      ),
      body: ...,
    )
```

### 如何从父View中Remove一个元素

答案是没有... 因为在Flutter看来, Widgets的树结构是不可以被更改的, 但是如果想更改, 则是通过StatefulWidgets的方法, 通过setState来更改Data, 触发Widgets重绘, 从而替换掉之前的Widgets.


### 喜欢画Canvas的同学怎么办?

Flutter同样支持, `CustomPaint`作为一个 Widgets就支持传入一个实现`CustomPainter`抽象类的参数, 而`CustomPainter`的抽象方法也类似于Android View的`onDraw`.

```
void paint(Canvas canvas, Size size)

bool shouldRepaint(CustomPainter oldDelegate)
```

### 如何自定义View

不用继承, 而使用类似Android ViewGroup的办法, 通过组合(composing)与封装的方法来实现, 通过小Widgets组合成需要的新Widgets.

### 页面跳转怎么办, 四大组件之一的Intent跑哪里去了

貌似在讲类似于Activity的`MaterialApp`的时候剧透了... 

就是使用`Navigator`与`Routes`来实现界面跳转, 实际上是整个Widgets的替换.

```
routes: <String, WidgetBuilder> {
      '/a': (BuildContext context) => MyPage(title: 'page A'),
      '/b': (BuildContext context) => MyPage(title: 'page B'),
      '/c': (BuildContext context) => MyPage(title: 'page C'),
    }
    
Navigator.of(context).pushNamed('/b');
```

### 如何处理外部的Intent

实际上还是需要在Flutter App的Android壳子中注册这个filter, 然后在FlutterActivity中拿到存下来, 

FlutterView初始化后再通过Bridge, 官方叫`MethodChannel`从Java里获取,进行下一步逻辑.

可以看个简单的例子.

```
new MethodChannel(getFlutterView(), "app.channel.shared.data").setMethodCallHandler(
      new MethodCallHandler() {
        @Override
        public void onMethodCall(MethodCall call, MethodChannel.Result result) {
          if (call.method.contentEquals("getSharedText")) {
            result.success(sharedText);
            sharedText = null;
          }
        }
      });
      
      
getSharedText() async {
    var sharedData = await platform.invokeMethod("getSharedText");
    if (sharedData != null) {
      setState(() {
        dataShared = sharedData;
      });
    }
  }
```

### 常用的startActivityForResult怎么办.

这个Flutter有完全对应的办法, 而且用起来很方便, 结合之前说的页面跳转:

```
Map xxx = await Navigator.of(context).pushNamed('/xxx');


Navigator.of(context).pop({xxx});

```

### 异步怎么办, runOnUiThread()哪里去了

Flutter有点像JS, 是一个单线程模式, 所以只是通过模拟来构建简单的异步, 关键字就是类似于kotlin coroutines一样, 通过`await`+`async`来处理.

如:

```
loadData() async {
    response = await http.get(xxx);
    setState(() {xxx});
}
```

但是由于它的单线程, 所以无法做很长的阻塞操作, 像http请求的延迟正常情况可能都是毫秒级的, 但是数据的处理等, 可能就得秒级了.

这也是RN在线程方面的做android程序的一个痛点, Flutter采用了比较容易想到的曲线救国的办法, 提供了一个叫`Isolate`的对象, 它实际是一个基于socket的数据通道, 相当于把数据放在一个独立的进程进行处理, 然后再通过socket发送回程序进程, 还记得进程间通信办法之一的`管道`吗...

具体API可以参考文档[1...](https://flutter.io/docs/get-started/flutter-for/android-devs),[2...](https://docs.flutter.io/flutter/dart-isolate/Isolate-class.html).

### Flutter 替代OkHttp的网络库

自带了http库, 直接`http.get(url)`, 在线程部分的代码实例里也有涉及.

通过类似gradle的文件`pubspec.yaml`引入.

```
dependencies:
  ...
  http: ^0.12
```

`^`表示不升大版本, 并取最新版本, 比gradle的+要范围更小.

### 常见的LCE(Loading Content Error)里面的Loading怎么show

Flutter有一个widget叫做`ProgressIndicator`, 比如我们期望有一个转圈圈的Loading界面在数据加载出来之前.

我们就可以通过StatefulWidgets, 根据数据, 或者List Widgets的个数 (如果是显示一个List的话)来判断是否显示Loading, 使用子类`CircularProgressIndicator`, 来替换页面的Widgets.

当然也是通过setState(() {...})来触发界面刷新的, 可以在initState()内触发加载数据的异步操作.

### 不同分辨率的图片资源怎么放

这个有点像iOS了, 即有1x,2x,3x:

```
images/my_icon.png       // Base: 1.0x image
images/2.0x/my_icon.png  // 2.0x image
images/3.0x/my_icon.png  // 3.0x image
```

不一样的一点还需要添加到类似gradle的文件`pubspec.yaml`里.

```
assets:
 - images/my_icon.jpeg
```

### 字符串怎么存储

Flutter没有像Android的`string.xml`的东西, 目前来说最好的就就是存成静态字符串.

```
class Strings {
  static String welcomeMessage = "Welcome To Flutter";
}

Text(Strings.welcomeMessage)
```

### Gradle变成什么了

前面说网络库, 图片资源的时候提到过, 提供了一个叫`pubspec.yaml`的文件, 具体支持的规则可以查看[这个文档](https://www.dartlang.org/tools/pub/pubspec).

### Fragment与Activity呢?

之前做过类比, 如`MaterialApp`有点类似于Activity, 而`Scaffold`都点类似Fragment, 实际上他们两个都是Flutter的Widgets, 也就是说其实只有View的概念了.

### 还有生命周期吗?

Flutter有一个叫做`WidgetsBinding`的可以提供类似生命周期的回调.

四种状态`inactive` (iOS专用), `paused`(相当于onPause, 退后台), `resumed`(相当于onPostResume, 到前台), `suspending`(android专用, 相当于onStop).

一般在StatefulWidgets的State中注册与反注册.

```
 @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addObserver(this);
  }

  @override
  void dispose() {
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }
```

### ScrollView vs ListView

Flutter没有ScrollView, 合并到了ListView, 通过ListView.builder创建的ListView提供了View复用的逻辑.

```
ListView.builder(
          itemCount: widgets.length,
          itemBuilder: (BuildContext context, int position) {
            return Text(xxx);
          }))
```

其中itemBuilder有点像Android ListView的getView, 官方文档说它会自动回收Element给你, 但是事实上每次你都需要根据position生成新的Widgets, 所以呢应该是Flutter在内部回收了之前的Widgets并在你重新创建的时候又用上了.

BTW, 通过ListView构造来显示就不具备这种特性, 所以大量数据需要用Builder.

### Flutter横竖屏怎么玩.

因为它实际上还是借助了Android程序的壳子, 所以如果AndroidManifect定义了`android:configChanges="orientation|screenSize"`, 则Flutter会自己hanlde.

### 怎么处理Gesture

Flutter提供了`GestureDetector`, 它相当于一个Container, 将我们期望接收手势的Widgets放进去, 再实现事件回调就行了.

```
GestureDetector(
        child: FlutterLogo(
          size: 200.0,
        ),
        onTap: () {
          print("tap");
        },
      )
```

它同样支持其他的手势, 如`onDoubleTap`等等等.

### 字体怎么弄

首先需要在`pubspec.yaml`里面配置需要的字体库:

```
fonts:
   - family: MyCustomFont
     fonts:
       - asset: fonts/MyCustomFont.ttf
       - style: italic
```

然后在Text的`style`属性进行配置.

```
Text(
        'This is a custom font text',
        style: TextStyle(fontFamily: 'MyCustomFont'),
      )
```

### Hint哪里去了, 错误信息怎么输出

对于输入框的Hint基本一致, 可能就是换了个名字, 一看便知.

```
TextField(
    decoration: InputDecoration(hintText: "This is a hint", errorText: _getErrorText()),
  )
```

## 总结

Flutter在视图渲染上另辟蹊径, 性能优势凸显, 在跨平台框架属于一匹黑马, 又有Google撑腰, 值得在Mobile勤耕多年的同学入手. 

由于作者曾经从事过2年的Webkit开发工作, 拜读了Flutter的渲染模式, 很像是Webkit/Chrome/Blink的思路, 通过查证, 起草者确实有大批同样的人, 如果你还没有入坑RN, 或许Flutter可以作为跨平台方案学习的首选哦. 

同样Google自己也有很多Plugin去支持更多扩展功能, 如GPS, Camera, SharePreference, Database. 还例如Firebase这种亲儿子级的服务也是全面支持Flutter. 这些都可以通过[Dartlang](https://pub.dartlang.org)来查询.

当然也可以自己去开发需要的Plugin来适配需要的功能, 基于的技术就是上面有提的`MethodChannel`, NDK的支持也是同样的道理.

## Reference

https://flutter.io/docs/get-started/flutter-for/android-devs