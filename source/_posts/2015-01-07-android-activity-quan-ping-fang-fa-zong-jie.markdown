---
layout: post
title: "Android Activity 全屏方法总结"
date: 2015-01-07 15:30:21 +0800
comments: true
categories: android
description: Android Activity 全屏方法总结
keywords: Android,Activity,全屏,fullscreen
tag: [android]
---
Android版本繁多, 新API, 新Flag层出不穷, 针对于如何完美全屏, 下面做以总结.

所有方法只涉及**代码**操作, 非xml修改Activity属性所致

<!--more-->

# 通用办法

如大家所知, 更改Window的属性, 并不一定在`setContentView`之前调用

```Java
    // FullScreen
    getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,
                            WindowManager.LayoutParams.FLAG_FULLSCREEN);
    // Cancel
    getWindow().setFlags(~WindowManager.LayoutParams.FLAG_FULLSCREEN,
                         WindowManager.LayoutParams.FLAG_FULLSCREEN);                           
```

此种办法可以应用于几乎所有版本(2.3, 4.x, 5.x), 但是在4.x上无法有隐藏动画效果

----------

从3.x版本开始, 系统`DecorView`提供了`setSystemUiVisibility`方法, 可以通过Flag改更改所谓SystemUI的属性

**下面介绍一下各种参数的意义**

    View.SYSTEM_UI_FLAG_VISIBLE Level 14  
    默认标记

    View.SYSTEM_UI_FLAG_LOW_PROFILE Level 14  
    低功耗模式, 会隐藏状态栏图标, 在4.0上可以实现全屏

    View.SYSTEM_UI_FLAG_LAYOUT_STABLE Level 16  
    保持整个View稳定, 常跟bar 悬浮, 隐藏共用, 使View不会因为SystemUI的变化而做layout

    View.SYSTEM_UI_FLAG_FULLSCREEN Level 16  
    状态栏隐藏

    View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN Level 16  
    状态栏上浮于Activity

    View.SYSTEM_UI_FLAG_HIDE_NAVIGATION Level 14  
    隐藏导航栏, 
    =========================================================================
    4.0 - 4.3如果使用这个属性,将会导致在下一次touch时候自动show出status跟navigation bar,源于系统clear掉其所有的状态
    =========================================================================

    View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION Level 16  
    导航栏上浮于Activity

    View.SYSTEM_UI_FLAG_IMMERSIVE Level 19  
    Kitkat新加入的Flag, 沉浸模式, 可以隐藏掉status跟navigation bar, 并且在第一次会弹泡提醒, 它会覆盖掉之前两个隐藏bar的标记, 并且在bar出现的位置滑动可以呼出bar

    View.SYSTEM_UI_FLAG_IMMERSIVE_STIKY Level 19  
    与上面唯一的区别是, 呼出隐藏的bar后会自动再隐藏掉

-------------
# 新方法

综合上述属性, 总结4.x以上版本全屏模式使用的最佳Flag组合(不希望悬浮可去除相关Flag, 并且用默认标记重新还原状态)

**4.0**
```Java
    getWindow().getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_LOW_PROFILE);
```

**4.1--4.3**  

隐藏导航会产生上面所说的问题,而FULLSCREEN已经可以将导航变成透明的小点点,达到一定的效果了

```Java
//Hide
getWindow().getDecorView().setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                        | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                        | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                        | View.SYSTEM_UI_FLAG_FULLSCREEN);
//Show
getWindow().getDecorView().setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                        | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                        | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN);
```

**4.4 above**  
```Java
//Hide
getWindow().getDecorView().setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                        | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                        | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                        | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION // hide nav bar
                        | View.SYSTEM_UI_FLAG_FULLSCREEN // hide status bar
                        | View.SYSTEM_UI_FLAG_IMMERSIVE);
//Show
getWindow().getDecorView().setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                        | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                        | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN);
```

**4.x综合办法**  

如果不希望4.4以上有IMMERSIVE特性,可删掉  

```Java
//Hide
getWindow().getDecorView().setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                        | View.SYSTEM_UI_FLAG_LOW_PROFILE
                        | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                        | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                        | View.SYSTEM_UI_FLAG_FULLSCREEN
                        | View.SYSTEM_UI_FLAG_IMMERSIVE);
//Show
getWindow().getDecorView().setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                        | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                        | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN);
```
-----------

  参考: 
    http://developer.android.com/reference/android/view/View.html#setSystemUiVisibility(int)  
    https://developer.android.com/training/system-ui/immersive.html