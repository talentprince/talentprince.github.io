---
title: Dagger2进阶必备技能
date: 2017-09-30 15:03:42
tags: [Dagger2,Android,Detect Inject,DI]
comments: true
description: 介绍Dagger2进阶使用,高级技巧
categories: Android
---

之前拙荆有一篇文章介绍Dagger2的[初步知识](http://www.cnblogs.com/mengdd/p/5613889.html), 本篇文章主要介绍Dagger2的进阶知识点.

主要包含的内有有

- @Binds与@Provides的使用
- Provider<T>与Lazy<T>的使用
- 依赖与包含
- Dagger.Android

<!--more-->

## @Binds与@Provides

相信大家经常会使用`@Provides`来在`Module`里面提供需要注入对象的构造, 但从来没有用过`@Binds`.

如果我们需要注入一个接口的实现,我们常常会这么做:

```
@Provides
public XXInterface providesXX(XXImp imp) {
  return imp;
}
```

其实这样的代码可以通过`@Binds`简化为

```
@Binds
public abstract XXInterface bindXX(XXImp imp);
```

同时你需要将你的`Module`改为`abstract`即可, 但是要注意这两个不能共存于一个Module,是不是很简单.

## Provider<T>与Lazy<T>

大家想必会使用过`Lazy`,很多语言都有`Lazy`,如最近大红大紫的`Kotlin`就可以通过`by lazy {}`来实现对象的延迟加载.

没错,`Lazy<T>`也是如此,只有当你调用`get()`时,才会真正注入.

`Provider<T>`与之的区别在于,`Lazy`延迟加载之后每次的调用都是同一个对象,而`Provider`则要看注入对象的实现,如果是通过`@Scope`约束的对象,则是同一个,否则每次都会创建新的.

## 依赖于包含

字面意思很好理解,翻译成Dagger的术语就是`dependency`与`subcomponent`.

下面分别举例子来说明其中的区别和使用方法.

#### Dependency

AnimalComponent
```
@Component(
        dependencies = FoodComponent.class,
        modules = AnimalModule.class
)
public interface AnimalComponent {
    Animal getAnimal();
}
```

AnimalModule
```
@Module
class AnimalModule {
  @Provides
  public Animal providesAnimal(Food food) {
    //Animal需要另外一个Component提供的Food来创建
    return new Animal(food);
  }
}
```

FoodComponent
```
@Component(modules = FoodModule.class)
public interface FoodComponent {
    //这个是关键,必须显示指出可以提供Food对象的生成
    Food getFood();
}
```

这样我们可以通过传入`FoodComponent`来完成注入, 如下:

```
DaggerAnimalComponent.builder().foodComponent(foodComponent).build().getAnimal()
```

#### Subcomponent

与依赖不同,`Subcomponent`拥有主`Component`所有注入对象,也就是说`Subcomponent`可以注入更多的对象, 通过生成代码也可以看出, 它的实现是主`Component`的内部类.

Cat
```
@Subcomponent(modules = {CatModule.class})
public interface CatComponent {
    Cat getCat();
}

```

CatModule
```
@Module
public class CatModule {
    @Provides
  public Cat providesCat(Leg leg//Animal Component提供) {
    return Cat(leg);
  }
}
```

我们还必须在AnimalComponent显示提供CatComponent,因为如上所述,Cat是Animal的内部类了.

```
@Component(
        dependencies = FoodComponent.class,
        modules = AnimalModule.class
)
public interface AnimalComponent {
    Animal getAnimal();
    CatComponent createCatComponent();
}
```

这样我们就可以通过下面的办法来实现`Cat`的注入:

```
DaggerAnimalComponent.build().createCatComponent().getCat();
```

#### Subcomponent with explicit builder

当我们`AnimalComponent`需要对Cat进行修改再输出的话(如指定猫的名字),可能就需要为`CatComponent`提供`Builder`

```
@Subcomponent(modules = {CatModule.class})
public interface CatComponent {
    @Subcomponent.Builder
    interface Builder {
        @BindsInstance Builder name(String name);
        CatComponent build();
    }
}
```

然后我们需要在`AnimalModule`里面使用这个`Builder`了

```
@Module(subcomponents = CatComponent.class)//注意这里需要加上这一条声明
class AnimalModule {
  @Provides
  public Animal providesAnimal(Food food) {
    //Animal需要另外一个Component提供的Food来创建
    return new Animal(food);
  }
  @Provides
  public CatComponent providesCatComponent(CatComponent.Builder builder) {
    //这里只是举个例子,可能这里的Cat构造依赖于Animal的另外属性
    return builder.name("喵喵").build();
  }
}
```

## Dagger.Android

平时我们注入的时候常常都是将`Component`存在`Application`里面,然后在Acitivity的`onCreate`或者Fragment的`onAttach`, 通过静态对象`Applicate.component.inject(xxx)`或者`((XXApplication)getApplication()).getComponent().inject(xxx)`来注入.

我们常常需要记住在固定的生命周期里面调用固定的语句,如果你的`Activity`或者`Fragment`在别的module里面使用公开的接口,对于`Fragment`你还可以对其对象进行注入(`inject(fragmentInstance)`),然而对`Activity`可能就没有很好的办法了...

Google的Dagger2提供了一套针对Android的东西,帮助你只需要调用`AndroidInjection.inject(this)`或者`AndroidSupportInjection.inject(this)`来注入,甚至还可以通过添加一些监听器,达到自动注入的效果哦.

下来看看如何实现吧

#### 添加依赖

```
//x>=10
implementation 'com.google.dagger:dagger-android:2.x'
// if you use the support libraries
implementation 'com.google.dagger:dagger-android-support:2.x'
annotationProcessor 'com.google.dagger:dagger-android-processor:2.x'
```

#### 引入`AndroidInjectionModule.class`到你的Component

#### 提供继承`AndroidInjector<T>`的`Subcomponent`, 及其`Builder`

```
@Subcomponent(modules = ...)
public interface YourActivitySubcomponent extends AndroidInjector<YourActivity> {
  @Subcomponent.Builder
  public abstract class Builder extends AndroidInjector.Builder<YourActivity> {}
}
```

#### 通过`Builder`提供对应`Activity`的`AndroidInjector.Factory`

```
@Module(subcomponents = YourActivitySubcomponent.class)
abstract class YourActivityModule {
  @Binds
  @IntoMap
  @ActivityKey(YourActivity.class)
  abstract AndroidInjector.Factory<? extends Activity>
      bindYourActivityInjectorFactory(YourActivitySubcomponent.Builder builder);
}

@Component(modules = {..., YourActivityModule.class})
interface YourApplicationComponent {}
```

#### Tips

类似于之前介绍`Subcomponent.Builder`, 如果你不需要定制该Builder, 如添加方法之类的, 那么这**上面两步可以做简化**.

```
@Module
abstract class YourActivityModule {
    @ContributesAndroidInjector(modules = { /* modules to install into, like FragmentModule */ })
    abstract YourActivity contributeYourActivityInjector();
}
```

参数`module`可以把想要注入的`Fragment`抽象到`YourFragmentModule`一并注入.

#### 实现`HasActivityInjector`

```
public class YourApplication extends Application implements HasActivityInjector {
  @Inject DispatchingAndroidInjector<Activity> dispatchingActivityInjector;

  @Override
  public void onCreate() {
    super.onCreate();
    DaggerYourApplicationComponent.create()
        .inject(this);
  }

  @Override
  public AndroidInjector<Activity> activityInjector() {
    return dispatchingActivityInjector;
  }
}
```

实际上所有的注入构造的工场方法`AndroidInjector.Factory`都被存入了一个`Map`保存在`DispatchingAndroidInjector`,我们可以查看其生成的代码,其中核心逻辑在`maybeInject(T instance)`里

```
public boolean maybeInject(T instance) {
    Provider<AndroidInjector.Factory<? extends T>> factoryProvider =
        injectorFactories.get(instance.getClass());
    if (factoryProvider == null) {
      return false;
    }
    AndroidInjector.Factory<T> factory = (AndroidInjector.Factory<T>) factoryProvider.get();
    try {
      AndroidInjector<T> injector = factory.create(instance);
      injector.inject(instance);
      return true;
    } catch (ClassCastException e) {
        ...
    }
  }
```

这就是其之所以能通过`AndroidInjection.inject(this)`实现注入的核心原理所在,`Fragment`同理.

当然你需要通过持有`AndroidInjector Module`的Component将这个`DispatchingAndroidInjector`注入了才行,一般可以在Application里面做.

#### 实现自动注入.

由于使用`Dagger.android`扩展使注入入口得到统一,那么就可以通过添加监听的方式在activity与fragment创建的时候实现自动注入.

当然相信之后此部分代码可能会被融入进`Dagger2`.

```
Application.registerActivityLifecycleCallbacks(new Application.ActivityLifecycleCallbacks() {
                    @Override
                    public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
                        handleActivity(activity);
                    }
                    ....
                }
                

private static void handleActivity(Activity activity) {
        if (activity instanceof HasSupportFragmentInjector) {
            AndroidInjection.inject(activity);
        }
        if (activity instanceof FragmentActivity) {
            ((FragmentActivity) activity).getSupportFragmentManager()
                    .registerFragmentLifecycleCallbacks(
                            new FragmentManager.FragmentLifecycleCallbacks() {
                                @Override
                                public void onFragmentCreated(FragmentManager fm, Fragment f,
                                        Bundle savedInstanceState) {
                                    if (f instanceof Injectable) {
                                        AndroidSupportInjection.inject(f);
                                    }
                                }
                            }, true);
        }
    }
```

#### 总结

`Dagger2.Android`自`2.10`版本后推出,只有不到三个月,可见`Dagger2`还在不断自我强大的过程中,它的出现使得`Android`开发在很多层面变的简单,如果有希望进一步学习的朋友,可以参考官方文档和一些Google的Sample,会在最后的`Reference`给出连接.

## Demo

[Github](https://github.com/talentprince/advanceddagger2)

## Reference

- https://google.github.io/dagger/android.html
- https://google.github.io/dagger/subcomponents.html
- https://github.com/googlesamples/android-architecture-components
