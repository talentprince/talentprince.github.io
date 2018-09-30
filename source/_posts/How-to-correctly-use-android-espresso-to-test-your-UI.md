---
title: 如何正确使用Espresso来测试你的Android程序
date: 2018-09-30 10:12:48
tags: [Android, Espresso, UI Automation Test]
categories: Android
comments: true
description: 如何使用Android Espresso测试UI
keywords: Android Espresso UI自动化 异步处理 IdlingResource Dagger
thumbnailImage: https://res.cloudinary.com/dtn0pkdmg/image/upload/v1538273751/espresso_wsc7bk.jpg
---

UI测试在Android平台上一直都是一个令人头痛的事情, 由于大家平时用的很少, 加之很多文档的缺失, 如果很多东西从头摸索,势必踩坑无数.

自Android24正式淘汰掉了[InstrumentationTestCase](https://developer.android.com/reference/android/test/InstrumentationTestCase)(位于android.test包), 推出[Espresso](https://developer.android.com/training/testing/espresso/)(位于android.support.test包), Google一直致力于降低UI测试的门槛.

了解测试金字塔的同学可能知道,UI测试属于功能测试(Functional Test), 或者按照其他的划分也属于集成测试(Integration Test), Google推出了[UIAutomator](https://developer.android.com/training/testing/ui-automator)与[Espresso](https://developer.android.com/training/testing/espresso)来分别处理跨App间的测试([黑盒测试](https://zh.wikipedia.org/wiki/%E9%BB%91%E7%9B%92%E6%B5%8B%E8%AF%95))以及App内的测试([白盒测试](https://zh.wikipedia.org/wiki/%E7%99%BD%E7%9B%92%E6%B5%8B%E8%AF%95)).

测试步骤类似,分为:

- 查找元素
- 触发行为
- 检测结果

本文分为三部分, 第一部分简单介绍如何使用Espresso, 第二部分分析如何处理诸如异步, 依赖注入, 程序结构对UI测试的影响以及提供解决办法, 第三部分提供源码以及一些Reference的地址.

<!--more-->

## Part I

### 如何配置

1.需要在gradle的dependencies里添加依赖

```
androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
androidTestImplementation 'com.android.support.test:runner:1.0.2'
androidTestImplementation 'com.android.support.test:rules:1.0.2'
```

2.在gradle的android.defaultConfig里指定TestRunner

```
testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
```

3.书写测试文件,通过`AndroidJUnit4`来跑即可,使用Activity Rule来启动你的Activity.


```
@Rule
@JvmField
var activityTestRule: ActivityTestRule<MainActivity> = ActivityTestRule<MainActivity>()
```

4.添加测试.

```
onView(withText("Hello world!")).check(matches(isDisplayed()));
```

5.运行

```
./gradlew connectedAndroidTest
```
或在IDE中进行运行.

以上步骤写的比较简略, 如果第一次使用, 可参考[官方文档](https://developer.android.com/training/testing/espresso/setup).

## Part II

### 貌似已经会了, 打钩[x]?

对于简单的UI其实上面的5步已经完全足够,  这也是Espresso好用的地方,  将UI测试写的跟普通的Unit Test一样简单.

但是随着你的UI变得复杂, 很多问题接踵而至.

其根本原因在于, Espresso系统在处理内置UI渲染(包括WebView)的异步操作都没有问题, 它会等待页面的渲染与加载, 而你自己如果有异步逻辑, 可能测试进程不会等待其完成而结束, 导致测试失败.

而采用Unit Test将无论是RxJava的Scheduler或者是Excutor替换成同一个线程的方法没法在UI Test中使用. 原因是UI操作只能在创建它的线程使用(UI 线程), 而如果你用了网络或者Room之类的数据库, 它又无法在UI线程使用, 相互矛盾, 进退两难.

所以这个时候就需要使用Espresso提供的[IdleResource](https://developer.android.com/training/testing/espresso/idling-resource), 来通知系统是否Idle或者Busy.

### 什么时候该使用IdleResource

其实IdleResource的官方文档里面有指出, 如果你的测试里有使用:

- Thread.sleep()
- Retry
- CountDown ...

来保证你的测试工作正常, 那么意味着你应该使用IdleResource了.

或许刚刚接触Espresso的你可能还没有意识到问题所在, 还没有使用Work Around的方法来解决问题, 换个角度来说可能更好理解.

如果你所测试程序里有使用:

- Databinding
- LiveData
- 通过非AsyncTask实现的异步操作
- Fragment跳转
- 等等...

那么就意味着你需要使用IdleResource来保证你的测试能顺利进行, 否则Test Case可能在程序异步操作未执行时就已经关闭了.


### 如何使用IdleResource

[IdleResource](https://developer.android.com/training/testing/espresso/idling-resource)的三个关键接口都非常Straigtforward.

1.
```
fun getName(): String
```

每一个IdleResource都应该有唯一的Name来注册到系统里, 不能重复.


2.
```
fun isIdleNow(): Boolean
```

Espresso会从UI线程调用, 通过这个方法来获得是否进入Idle状态.


3.
```
fun registerIdleTransitionCallback(callback: IdlingResource.ResourceCallback)
```

当该IdleResource被使用时, Espresso会注册该callback, 当background job执行完毕后, 需要调用callback.onTransitionToIdle()通知(**如果已经是Idle状态, 调用也不影响, 所以很多简单的实现都是将这个调用放在isIdleNow中, 判断已经idle就调用, 虽然google的best practice里说不要这样**), 该调用会通知UI线程, 并可以在任何线程调用.



在使用IdleResource的时候, 通常是通过注册Rule来驱动的, 这个就需要继承`TestWatcher`.

复写它的starting与finished方法, 通过`IdlingRegistry.getInstance().register`与`IdlingRegistry.getInstance().unregister`来注册/反注册IdleReource, 当然可能需要在finished的时候drain掉所有在运行的Task.

给一个简单的例子把.

```
class SampleIdleResourceRule : TestWatcher() {
    private val idlingResource: IdlingResource = xxx
    
    override fun starting(description: Description?) {
        IdlingRegistry.getInstance().register(idlingResource)
        super.starting(description)
    }
    
    override fun finished(description: Description?) {
        //drain all the pending task here if needed.
        IdlingRegistry.getInstance().unregister(idlingResource)
        super.finished(description)
    }
}
```

### 举个IdleResource的例子吧.

1.使用LiveData等Archtecture Component组件

我们知道LiveData是一个订阅系统, 是必涉及后台线程, 比较方便的是它自己内部已经调用了IdleResource来增加/减少后台job, 所以直接使用系统提供的[CountingTaskExecutorRule](https://developer.android.com/reference/android/support/test/espresso/idling/CountingIdlingResource).

由于Resource name不能重复, 所以为了绕过这个检测, 需要继承`CountingTaskExecutorRule`来复写getName.

具体可以参考google的[TaskExecutorWithIdlingResourceRule](https://github.com/googlesamples/android-architecture-components/blob/1c91038b55d52fe1006b3b4c6436003f4da29c4f/GithubBrowserSample/app/src/androidTest/java/com/android/example/github/util/TaskExecutorWithIdlingResourceRule.kt#L31).

Google还提供了Databinding的Rule, 可以参考.

2.等待弹框结束

一般情况下我们使用DialogFragment来弹框, 如果我们去check一些text被dialog遮挡, 就必须等待其消失后在进行检查.

这时我们可以通过`findFragmentByTag`来检测该弹框是否dismiss.

```
class DialogIdlingResource(
        private val manager: FragmentManager, 
        private val tag: String) : IdlingResource {
    private var resourceCallback: IdlingResource.ResourceCallback? = null

    override fun getName(): String = "xxx"

    override fun registerIdleTransitionCallback(callback: IdlingResource.ResourceCallback?) {
        resourceCallback = callback
    }

    override fun isIdleNow(): Boolean {
        val idle = manager.findFragmentByTag(tag) == null
        if (idle) {
            resourceCallback?.onTransitionToIdle()
        }
        return idle
    }
}
```

3.Delegate Executors/Scheduler

如果有异步处理逻辑, 大多都位于Repository/ViewModel层, 这部分会被Mock, 但也有一些UI逻辑可能会用到Excecutor. 如RecyclerView的DiffUtil, 需要传入一个Executor来做异步Diff, 这时我们就需要一个Excecutor的IdlingResource, 并把它里面的Delegate赋值给UI.

这部分可以参考Google GithubBrowser Sample的[CountingAppExecutorsRule](https://github.com/googlesamples/android-architecture-components/blob/1c91038b55d52fe1006b3b4c6436003f4da29c4f/GithubBrowserSample/app/src/androidTest/java/com/android/example/github/util/CountingAppExecutorsRule.kt#L32).


### 应该怎么测试, 需要测试什么?

虽然Espresso测试是集成测试,  但是由于涉及到异步逻辑导致Test Case无法按照预期进行的问题时而存在, 且有时候无法通过IdlingResource来解决.

比如涉及到多个Fragment的跳转, 就会发生在Fragment未打开时Test Case就挂掉的情况.

再比如使用RxJava, 在Espresso3.x + RxJava2.x的情况下, 即便将Scheduler代理给IdlingResource也无法保证整个业务流程完整走下来, 异步操作仍无法完整运行, 具体问题可参考Jake大神RxIdler的[Issue](https://github.com/square/RxIdler).

所以测试起来就有一些原则需要遵守, 才能保证整个流程的可测性.

- 最好对每一个Fragment进行单独测试, Mock所依赖的部分, 如网络, 数据模块, 如果涉及Fragment跳转逻辑, 通过继承来复写进行测试.
- 如果使用了RxJava, 需要将其封装在Repository或者Presenter/ViewModel中进行整体的Mock.
- 如果使用了Dagger2.android进行自动注入, 最好对测试部分自定义TestRunner提供一个空的Application来Disable注入, 对所测试Fragment注入对象进行手动赋值.
- 如果Activity有注入逻辑, 最好将其解耦到Fragment, 因为Espresso的Activity是通过ActivityRule来启动, 无法进行直接手动注入.
- 如果无法Move到Fragment, 或者不想... 那就需要在测试里构建自己的Dagger Component, 对于使用Dagger2.android自动注入的, 还需要手动创建Fake的DispatchingAndroidInjector完成手动注入.
- 如果未使用Dagger2.android, 通过AndroidInjector来注入的, 可以忽略与注入相关的item.


### 能再讲的仔细一些吗?

1.单独测试Fragment的好处是可以解耦Fragment之间的跳转, 往往Fragment都是UI流程中的一个环节, 当逻辑完成时会跳向下一Fragment. 可以创建一个空Activity来专门用于显示该Fragment, 并且在测试的setUp里commit该Fragment.


```
class TestActivity {
    fun showFragment()
}

@RunWith(AndroidJUnit4::class)
class XXXFragmentTest {
    @Rule
    @JvmField
    val activityRule = ActivityTestRule(XXXFragment::class.java)
    
    @Before
    fun init() {
        //1. init fragment
        //2. assign mock data
        activityRule.showFragment(xxx)  
    }
    
    @Test
    fun testXXX() {
        xxx
    }
}
```

2.由于常常会需要继承需要测试的Fragment来复写一些类, 对于使用Dagger.android自动注入的, 该子Fragment又未通过`@ContributesAndroidInjector`进行注册, 往往需要自定义TestRunner, 然后手动注入Fragment.


```
class CustomTestRunner : AndroidJUnitRunner() {
    override fun newApplication(...) {
        return ...TestApp:class...
    }
}

android {
    defaultConfig {
        testInstrumentationRunner "xxx.CustomTestRunner"
    }
}

class TestApp : Application() {}

@RunWith(AndroidJUnit4::class)
class XXXFragmentTest {
    //activity rule
    ...
    val testFragment = TestFragment()
    
    @Before
    fun init() {
        testFragment.xxx = mockXXX
        ...
        activityRule.activity.showFragment(testFragment)
    }
    
    @Test
    fun testXXX() {
        onView...check(...)
        assertTrue(testFragment.isXXXShow)
    }
    
    class TestFragment : XXXFragment() {
        var isXXXShow = false
        override fun showXXX() {
            isXXXShow = true
        }
    }
}

```

3.如果Activity有注入逻辑与业务逻辑, 并且不想抽到Fragment中去, 则需要创建Fake的Injector保证可以完成注入, 
 
```
fun createFakeInjector(block: T.() -> Unit): DispatchingAndroidInjector<Activity> {
    ...
}

@RunWith(AndroidJUnit4::class)
class XXXActivityTest {
    @Rule
    @JvmField
    var activityRule = object : ActivityTestRule<XXX>(XXX::class.java) {
       val app = ...get application
       app.dispatchingAndroidInjector = createFakeInjector<XXX>() {
           //手动注入
           xxx =  mockXXX
           `when`(xxx).thenReturn(xxx)
       }
    }
}
```


4.为了支持需要通过继承Fragment来完成测试的Case, 还需要对测试模块创建自己的Component来注册从而进行Fake Injector的创建 (类似3, 只是Application/Activity可能为Test版本).


```
Grale

dependencies {
    kaptAndroidTest 'com.google.dagger:dagger-android-processor:2.X'
}


@Component(modules = [
    AndroidInjectionModule::class,
    AndroidSupportInjectionModule::class,
    ...主App所注册的所有Module,
    TestActivityModule::class])
interface TestCompnent {
    fun inject(xxx: XXX)
    ...
}

@Module
abstract class TestActivityModule {
  //通过`ContributesAndroidInjector`注册你的TestActivity, 以及TestFragment  
}

class TestApp : Application(), HasActivityInjector {
    @Inject
    lateinit var injector: DispatchingAndroidInjector<Activity>
}

class TestActivity : Activity(), HasSupportFragmentInjector {
    @Inject
    lateinit var injector: DispatchingAndroidInjector<Fragment>
}
```

## Part III

### 如果还不是很明白可以查看代码

Disable注入的在这里:
[Google的Demo GithubBrowser](https://github.com/googlesamples/android-architecture-components/tree/master/GithubBrowserSample)

跟注入相关的在这里:
[自己的Demo](https://github.com/mengdd/DribbbleClient)


### Reference 

- https://proandroiddev.com/activity-espresso-test-with-daggers-android-injector-82f3ee564aa4
- https://github.com/SabagRonen/dagger-activity-test-sample
- https://github.com/googlesamples/android-architecture-components/tree/master/GithubBrowserSample
- https://developer.android.com/training/testing/espresso/idling-resource
- http://blog.sqisland.com/2015/07/espresso-wait-for-dialog-to-dismiss.html


