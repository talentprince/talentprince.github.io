---
layout: post
title: "Android 构建账号与同步服务 Part One"
date: 2015-01-26 08:33:37 +0800
comments: true
categories: Android
Tags: [android]
keywords: Android,SyncAdapter,Account,Authenticator,账号,同步,后台更新
description: Android 构建账号与同步服务 SyncAdapter Account Authenticator
---

####账号与同步

Android从`API Level5`就有了自己的同步服务, 但很少有程序使用到, 一来大多数程序不需要所谓的同步,二来很多程序自己实现了后台的同步更新. 随着Android程序开发的逐渐程序, 越来越的的程序使用到了系统提供的服务来完成`账号认证`与`同步更新`, 我们可以打开系统设置-->账号进行查看, 就能看到很多应用都这么做了. 这样做有两个好处, 一来系统服务做更新同步(`SyncAdapter`)唤醒更加绿色环保, 二来实现了账号认证(`Authenticator`)还可以为其他应用提供第三方认证服务, 如大家常见的使用QQ或者微博账号登录, 由于你手机上安装的QQ与微博实现了该接口, 便可以通过开发者账号获得授权Token来做第三方认证.

本期博客分三部分来讲, 通过一个小应用(Part Three提供源码)来概述所有相关内容, 大体章节如下

* 数据模型建立与加载 (ContentProvider LoaderManager)

* 更新系统建立 (SyncAdapter)

* 账号系统建立 (Account Authenticator)

下面先来讲讲如何轻松本地数据库并完成数据到界面的加载

<!--more-->

#####数据模型建立

数据同步与Android四大组件之一`Content Provider`紧密相关, 所以我们的存储需要实现自己的Content Provider, 在这里推荐使用一个Generator, 可以迅速生成出自己想要的Content Provider, 并提供非常便捷的操作, 详细操作可见[Bod](https://github.com/BoD/android-contentprovider-generator), 当然这个工具生成的数据库有一些弊端, 比如主键只能是_id, 并且还不能操作, 只支持一个外键等等

我们的App是一个简单的记事本, 数据库Column有`title` `create_time` `update_time` `content`四列

使用[Bod](https://github.com/BoD/android-contentprovider-generator)主要需要定义`_config.json`与`xxx_table.json`文件

下面可以看看我们App是怎么定义的

```groovy _config.json
{
    "syntaxVersion": "2",
    "projectPackageId": "org.weyoung.notebook",
    "authority": "org.weyoung.notebook.provider",
    "providerJavaPackage": "org.weyoung.notebook.data",
    "providerClassName": "NotebookProvider",
    "sqliteOpenHelperClassName": "NoteBookSQLHelper",
    "sqliteOpenHelperCallbacksClassName": "NoteBookSQLHelperCallbacks",
    "databaseFileName": "notebook.db",
    "databaseVersion": 1,
    "enableForeignKeys": true,
    "useAnnotations": true,
}
```

```groovy notedata.json
{
    "fields": [
        {
            "name": "title",
            "type": "String",
            "nullable": "false",
            "index": true,
        },
        {
            "name": "create_time",
            "type": "String",
            "nullable": "false",
        },
        {
            "name": "update_time",
            "type": "String",
            "nullable": "false",
        },
        {
            "name": "content",
            "type": "String",
            "nullable": "true",
        },
    ],
    "constraints": [
        {
            "name": "unique_title",
            "definition": "UNIQUE (title) ON CONFLICT REPLACE"
        },
    ]
}
```

一目了然, 运行`java -jar android_contentprovider_generator-1.8.4-bundle.jar -i [input] -o [output]`便可以得到我们想要的东西啦, 并将`Log`里面输出的`Provider`声明复制到`AndroidManifest.xml`里面

```xml
<provider
    android:name=".data.NotebookProvider"
    android:authorities="org.weyoung.notebook.provider"
    android:exported="false"/>
```

#####数据模型加载

这里我们使用Android提供的[LoaderManager](https://developer.android.com/training/load-data-background/setup-loader.html), 它本身也是一个`Observer`, 只要`Loader`对应的`URI`的`ContentProvider`发生变化(`notifyChanged`), 其`Callback`的`onLoadFinished`都会获得对应的数据.

首先必须实现`LoaderManager.LoaderCallbacks<Cursor>`的方法, 并且初始化`Loader`.

代码如下

```Java
//初始化Loader, 传入callback
getLoaderManager().initLoader(0, null, callback);

//实现对应的callback
@Override
    public Loader<Cursor> onCreateLoader(int id, Bundle args) {
        //创建读取Notedata的Loader, NotedataColumns为BoD生成的类
        CursorLoader cursorLoader = new CursorLoader(this);
        cursorLoader.setUri(NotedataColumns.CONTENT_URI);
        return cursorLoader;
    }

    @Override
    public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
        List<Note> notes = new ArrayList<>();
        //使用BoD生成的ContentProvider工具类来解析数据, 非常方便
        NotedataCursor cursor = new NotedataCursor(data);
        while(cursor.moveToNext()) {
            notes.add(new Note(cursor.getTitle(), cursor.getCreateTime(), cursor.getUpdateTime(), cursor.getContent()));
        }
        //更新UI
        noteRecyclerAdapter.setNotes(notes);
        noteRecyclerAdapter.notifyDataSetChanged();
    }

    @Override
    public void onLoaderReset(Loader<Cursor> loader) {

    }
```

只要对`NotedataColumns.CONTENT_URI`的Provider进行操作, `onLoadFinished`都会被回调, 这就轻松的完成了对数据的监听, 下面只剩显示到界面上了

#####数据显示

可以通过上面看出, 这里使用了新出的`RecycleView`, 当然也可以使用`ListView`配合`CursorAdapter`来显示, 那就顺便介绍下[RecycleView](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html)怎么使用吧.

这里我们使用兼容包的View以获得对5.0以下的支持.

添加gradle

```groovy gradle
    compile 'com.android.support:recyclerview-v7:21.0.3'
```

添加布局文件

```xml layout
    <android.support.v7.widget.RecyclerView
        android:id="@+id/notes"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
```

实现`RecyclerViewAdapter`

```Java
public class NoteRecyclerAdapter extends RecyclerView.Adapter<NoteViewHolder> {
    List<Note> notes = new ArrayList<>();

    public void setNotes(List<Note> notes) {
        this.notes = notes;
    }

    @Override
    public NoteViewHolder onCreateViewHolder(ViewGroup viewGroup, int i) {
        final Context context = viewGroup.getContext();
        View view = LayoutInflater.from(context).inflate(R.layout.note, null);
        return new NoteViewHolder(view);
    }

    @Override
    public void onBindViewHolder(NoteViewHolder noteViewHolder, int i) {
        noteViewHolder.populate(notes.get(i));
    }

    @Override
    public int getItemCount() {
        return notes.size();
    }
}
```

这里可以看出来其已经自己完成了对View的复用优化, 并且提供了统一个`Holder`类来包装`View`操作

定义自己的`ViewHolder`

```Java
public class NoteViewHolder extends RecyclerView.ViewHolder {
    @InjectView(R.id.title)
    TextView title;
    @InjectView(R.id.create_time)
    TextView createTime;
    @InjectView(R.id.modify_time)
    TextView modifyTime;
    private View view;

    public NoteViewHolder(View itemView) {
        super(itemView);
        view = itemView;
        ButterKnife.inject(this, itemView);
    }

    public void populate(final Note note) {
        //渲染UI
    }
}
```

完成`RecyclerView`配置

```Java
    noteRecyclerAdapter = new NoteRecyclerAdapter();
    recyclerView.setLayoutManager(new LinearLayoutManager(this));
    recyclerView.setAdapter(noteRecyclerAdapter);
```

这里看以看到RecyclerView需要指定`LayoutManager`, 它支持`LinearLayout` `GridLayout`布局方式, 并且可以通过`addItemDecoration`自己渲染每一个Item的装饰(divider 等等), 更炫酷的是它可以通过`setItemAnimator`设置Item添加删除动画, **动画需要通过Adapter的notifyItemInserted与notifyItemRemoved**触发.

--------------

这样就完成了数据的存储与显示了, 里面用到了[之前](http://talentprince.github.io/blog/2015/01/04/android-yu-inject-yi-lai-zhu-ru-de-bu-jie-zhi-yuan/)提到的[Butterknife](http://jakewharton.github.io/butterknife/)完成View的注入等简单操作.

下一部分, 将介绍备受关注的更新系统`SyncAdapter`的使用

----------

参考:
https://developer.android.com/training/load-data-background/setup-loader.html

https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html

https://github.com/BoD/android-contentprovider-generator






