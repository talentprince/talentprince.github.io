---
layout: post
title: "Android 与Inject(依赖注入)的不解之缘"
date: 2015-01-04 14:56:04 +0800
comments: true
categories: Android
description: Android 与Inject(依赖注入)的不解之缘
keywords: Android Inject Butterknife Dagger
tags: [android, inject]
---
 依赖注入(DI)
 有些人说Android使用依赖注入是因为很多J2EE的人带来的异域思想, 满天飞的`注解`让人莫不找头脑, 使简单的行为变得复杂, 表面简化, 实则复杂.

 但是在使用其一段时间后, 确实还是挺不错的. 正如其思想之精髓, 让你只关注**结果**,而忽略**制作过程**, 呵呵, 跟`周星驰`他老母恰巧相反.

 那么下面就讲讲Android开发中常常的用的一些DI框架, 来简化亲们的开发流程吧.

 <!--more-->

----------------

#Butterknife
Butterknite是一个专注于简化View相关使用的DI框架, 而且其不像`Spring`一样使用反射, 而通过预编译来做到注入效果, 大大降低了消耗, 绿色环保, 下面就来讲讲他的使用, 其实很简单.

配置:
由于目前基本都是用Android Studio开发, 这里添加gradle的依赖就行了  
```
    compile 'com.jakewharton:butterknife:6.0.0'
```

#使用方法:
1. 初始化注入  
```Java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //初始化
        ButterKnife.inject(this);

    }
```

2. 指定控件的id, 这里定义了 `TextView` `EditText` `Button` 用于测试

3. 使用`Butterknife`注解简化代码  
```Java
    //声明注入的View
    @InjectView(R.id.search_src_text)
    TextView textView;
    @InjectView(R.id.edit_query)
    EditText editText;
    @InjectView(R.id.search)
    Button button;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.inject(this);

        //可直接使用啦
        textView.setText("Hehe");
    }

    //也可以注入Listener
    @OnClick(R.id.search)
    void setEditText() {
        editText.setText("Haha");
    }
``` 

这里可以看出, 以往通过`findViewById`以及`setOnClickListener`都得到了简化, 我们**冲上去**使用就行了

类似的还有其他的注解, 如`@OnLongClick` `@OnItemClick` `@OnItemSelected`, 用法类似
当layout复用的时候, 有些不一定存在的View可以通过`@Optional`来标识, 否则会**爆炸**

详细使用可参照 http://jakewharton.github.io/butterknife/

----------

#Dagger
说简单一些, 使用Dagger注入依旧绿色环保, 省去了日常所关注的`构造`过程, 让所有指定在`Moudle`里面的类型, 想用即用.

Dagger的工作原理  
![Dagger](/images/res/201501/dagger.png)

配置:  
```
    compile 'com.squareup.dagger:dagger:1.2.2'
    compile 'com.squareup.dagger:dagger-compiler:1.2.2'
```

#使用方法  

1.定义Moudle (告诉Dagger其如何构造)  
我们首先先定义一个Android服务相关的Moudle, 简化我们对`LocationManager`的使用  
```Java AndroidModule
    @Module(library = true,
            complete = false)
    public class AndroidModule {
      @Provides 
      @Singleton 
      LocationManager provideLocationManager(Application application) {
        return (LocationManager) application.getSystemService(LOCATION_SERVICE);
      }
    }
```

这里的`library = true`的意思是可能不会被使用, 这只是一个lib, 默认为false, 如没有被使用会抛错  
`complete = false`的意思是这是一个不完整的moudle, 为何这么说, 可以看出`provideLocationManager`的参数没有响应提供值的`Providers`呢, 为什么呢, 因为这个Moudle接下来要被另外一个Moudle引用, 所以application这个参数我们将在下一个Moudle里提供.  
`@Provides`是必须要有的注解, 告诉Dagger这个方法就是提供`LocationManager`的方法.`@Singleton`是标注这是一个单例, 保证了线程安全.  

```Java DemoMoudle
@Module(
    //声明将要使用注入对象的类
    injects = DemoActivity.class,
    includes = AndroidModule.class,
    library = true
)
public class DemoModule {
    private Context application;

    public DemoModule(Context application) {
        this.application = application;
    }

    @Provides
    @Singleton
    Context provideApplicationContext() {
        return application;
    }

    @Provides
    @Named("hehe")
    DaggerYa provideDaggerYa1(Context context) {
        return new DaggerTa(context, "hehe");
    }

    @Provides
    @Named("haha")
    DaggerYa provideDaggerYa2(Context context) {
        return new DaggerTa(context, "haha");
    }

    @Provides
    DaggerLa providerDaggerLa(Context context,
                              @Named("hehe")
                              DaggerYa  daggerYa) {
        return new DaggerLa(context, daggerYa);
    }
}
```

这个Moudle比较长, 首先使用了`include`来包含我们之前定义的Moudle.
而且也可以发现,这个Moudle提供了`Context`的`Provides`

`injects`比较重要, 这里指定的`class`就是那些需要使用注入对象的类

有些同学可能会有同样类型的对象注入的问题, 问题的解决办法就是使用`@Named`来标识其使用的是哪一个`@Provides`.

2.将Moudle提供给容器`ObjectGraph`(把之前定义的各个对象的构造方法交给Dagger来维护, 它将会把其注入到程序中去)  
    由于**Application**一个Android程序只有一个, 可以通过`getApplication`获得,那么最佳存放容器与注入的地方就是这里了,请看代码
```Java DemoApplication
public class DemoApplication extends Application {
  private ObjectGraph graph;

  @Override public void onCreate() {
    super.onCreate();

    graph = ObjectGraph.create(getModules().toArray());
  }

  protected List<DemoModule> getModules() {
    return Arrays.asList(new DemoModule(this));
  }

  public void inject(Object object) {
    graph.inject(object);
  }
}
```

3.使用注入对象  
劳烦了半天, 终于可以尝尝甜头了, 下面看看使用的办法, 请看代码
```Java DemoActivity
public class DemoActivity extends Activity {
    @Inject
    LocationManager locationManager;
    @Inject
    @Named("hehe")
    DaggerYa daggerYa;
    @Inject
    DaggerLa daggerLa;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        //注入
        ((DemoApplication) getApplication()).inject(this);
    }
}
```

使用注入对象的类, 定义的Moudle里面必须要有这个类的声明(`injects = xxx.class`), 并且需要调用容器的`inject`方法(如加粗部分)

下面就可以通过`@Inject`轻松使用**无需再构造**的对象了, 以后只需要添加`injects`, 并且调用`inject`方法, 就可以在其他类继续**无构造**使用提供过`Provides`的对象啦

--------------

详细可参考: http://square.github.io/dagger/



