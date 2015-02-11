---
layout: post
title: "Android 构建账号与同步服务 Part Three"
date: 2015-02-10 17:13:33 +0800
comments: true
categories: Android
Tags: [android]
keywords: Android,SyncAdapter,Account,Authenticator,账号,同步,后台更新
description: Android 构建账号与同步服务 SyncAdapter Account Authenticator
---

####紧接[上期](http://talentprince.github.io/blog/2015/02/04/android-gou-jian-zhang-hao-yu-tong-bu-fu-wu-part-two/)

####账号系统建立 Account Authenticator

如果只需要借助系统更新服务(SyncAdapter)来做定期维护, 那么通过前两部分的介绍, 已经可以达到所预期的目标了.

本期话题将会解决

* 添加账号
* 获得授权

这些服务全部都是可以跨进程的操作, 完成了这些操作, 我们就可以完成像`QQ` `新浪微博` 一样的功能, 账号系统可以为第三方应用授权.

<!--more-->

####实现AccountService 以及 Authenticator

先看看所提供的重写方法

```Java
public class AccountService extends Service {

    private Authenticator authenticator;
    public static final String ACCOUNT_TYPE = "org.weyoung.notebook";
    public static final String TOKEN_TYPE = "org.weyoung.notebook.admin";

    @Override
    public void onCreate() {
        authenticator = new Authenticator(this);
    }
    @Override
    public IBinder onBind(Intent intent) {
        return authenticator.getIBinder();
    }


    class Authenticator extends AbstractAccountAuthenticator {
        private Context context;
        private AccountManager accountManager;

        public Authenticator(Context context) {
            super(context);
            this.context = context;
            accountManager = AccountManager.get(context);

        }

        @Override
        public Bundle editProperties(AccountAuthenticatorResponse accountAuthenticatorResponse,
                                     String s) {
            throw new UnsupportedOperationException();
        }

        @Override
        public Bundle addAccount(AccountAuthenticatorResponse response, String accountType, String authTokenType, String[] requiredFeatures, Bundle options)
                throws NetworkErrorException {
            //添加账号
        }

        @Override
        public Bundle confirmCredentials(AccountAuthenticatorResponse accountAuthenticatorResponse,
                                         Account account, Bundle bundle)
                throws NetworkErrorException {
            return null;
        }

        @Override
        public Bundle getAuthToken(AccountAuthenticatorResponse response, Account account, String s, Bundle bundle)
                throws NetworkErrorException {
            //获取认证
        }

        @Override
        public String getAuthTokenLabel(String s) {
            throw new UnsupportedOperationException();
        }

        @Override
        public Bundle updateCredentials(AccountAuthenticatorResponse accountAuthenticatorResponse,
                                        Account account, String s, Bundle bundle)
                throws NetworkErrorException {
            throw new UnsupportedOperationException();
        }

        @Override
        public Bundle hasFeatures(AccountAuthenticatorResponse accountAuthenticatorResponse,
                                  Account account, String[] strings)
                throws NetworkErrorException {
            throw new UnsupportedOperationException();
        }
    }
}
```

通过`Authenticator`所提供的方法可以看出, 最核心的就是实现`addAccount`与`getAuthToken`方法, 这样我们的`AccountService`就可以获得**授权**与**添加账号**功能了

####添加账号

先看代码

```Java
addAccount() {
    final Bundle bundle = new Bundle();
    //限制绑定账号只能有一个
    if(accountManager.getAccountsByType(ACCOUNT_TYPE).length > 0) {
        final Intent intent = new Intent(context, FailActivity.class);
        intent.putExtra(AccountManager.KEY_ACCOUNT_AUTHENTICATOR_RESPONSE, response);
        bundle.putParcelable(AccountManager.KEY_INTENT, intent);
        return bundle;
    }
    final Intent intent = new Intent(context, AuthenticatorActivity.class);
    intent.putExtra(AccountManager.KEY_ACCOUNT_AUTHENTICATOR_RESPONSE, response);   

    bundle.putParcelable(AccountManager.KEY_INTENT, intent);
    return bundle;
}
```

这里返回的`addAccount`返回的Bundle必须带有一个Key为**AccountManager.KEY_INTENT**, 并且这个`Intent`含有一个Key为`AccountManager.KEY_ACCOUNT_AUTHENTICATOR_RESPONSE`, Value为参数`AccountAuthenticatorResponse`. 这个response主要用来回调成功与否, 内部还可以传递生成账号的信息(Name,Type,Token), 将会在启动的用于注册的`AuthenticatorActivity`里使用.

关于`AuthenticatorActivity`的实现, 其实很简单, 其继承`AccountAuthenticatorActivity`, 这样父类自己帮你在`onCreate`的时候获取到了之前传入的`AccountAuthenticatorResponse`, 然后在注册成功后调用`setAccountAuthenticatorResult`来传出一个Bundle, 内部其实调用了前面所指的response对象的`onResult`或者`onError`方法.

这个Bundle可以含有若干重要的字段, `AccountManager.KEY_ACCOUNT_NAME` `AccountManager.KEY_ACCOUNT_TYPE` `AccountManager.KEY_AUTHTOKEN`

下面看看`AuthenticatorActivity`的核心实现吧.

```Java
submit() {
    final Account account = new Account(name, AccountService.ACCOUNT_TYPE); 

    //添加账号到系统Account中心, 这里输入的password可以做转换之类的
    ......认证 生成token 的逻辑
    accountManager.addAccountExplicitly(account, password, null);
    //添加token
    accountManager.setAuthToken(account, AccountService.TOKEN_TYPE, authToken); 

    //返回信息, 可以供getAuthToken使用
    Bundle data = new Bundle();
    data.putString(AccountManager.KEY_ACCOUNT_NAME, name);
    data.putString(AccountManager.KEY_ACCOUNT_TYPE, AccountService.ACCOUNT_TYPE);
    data.putString(AccountManager.KEY_AUTHTOKEN, authToken);
    Intent intent = new Intent();
    intent.putExtras(data);
    //如果不设置, 相当于调用response的onCancel取消注册
    setAccountAuthenticatorResult(data);
    finish();
```

####获得认证

先看代码

```Java
getAuthToken() {
    //获得一个已存在的token
    String authToken = accountManager.peekAuthToken(account, TOKEN_TYPE);   

    //如果没有可能过期了 拿到密码 重新认证一次
    if (TextUtils.isEmpty(authToken)) {
        final String password = accountManager.getPassword(account);
        if (password != null) {
            try {
                authToken = 生成token的逻辑;
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }   

    //如果有token了, 传递
    if (!TextUtils.isEmpty(authToken)) {
        final Bundle result = new Bundle();
        result.putString(AccountManager.KEY_ACCOUNT_NAME, account.name);
        result.putString(AccountManager.KEY_ACCOUNT_TYPE, account.type);
        result.putString(AccountManager.KEY_AUTHTOKEN, authToken);
        return result;
    }   

    //如果没有token, 也没有密码, 需要重新登录, 账户名可一并传递进认证的Activity, 如果有的话
    final Intent intent = new Intent(context, AuthenticatorActivity.class);
    intent.putExtra(AccountManager.KEY_ACCOUNT_AUTHENTICATOR_RESPONSE, response);
    intent.putExtra(AuthenticatorActivity.ARG_ACCOUNT_NAME, account.name);
    final Bundle b = new Bundle();
    b.putParcelable(AccountManager.KEY_INTENT, intent);
    return b;
}
```

由上面的代码可以看见, 对于需要获取token(getAuthToken)的操作, 我们的步骤是从简到繁, 最差情况就是需要重新的登录认证来获得token

跟添加账号相同, 最终账号的信息(Name Type Token)都是通过神奇的`Bundle`来传递的

####声明账号服务

与同步服务一样, 账号服务同样需要声明, 以便通过系统的Accout功能操作, 或者为其他第三方程序提供支持.

```xml
<service
    android:name=".sync.AccountService"
    android:exported="true"
    android:process=":auth" >
    <intent-filter>
        <action android:name="android.accounts.AccountAuthenticator" />
    </intent-filter>
    <meta-data
        android:name="android.accounts.AccountAuthenticator"
        android:resource="@xml/authenticator" />
</service>
```

这里跟[上一节](http://talentprince.github.io/blog/2015/02/04/android-gou-jian-zhang-hao-yu-tong-bu-fu-wu-part-two/)的`SyncService`类似, 必须声明为`exported`, `process`不是必须的, 并且需要添加`android`约定的过滤器**android.accounts.AccountAuthenticator**

关于`Authenticator`的配置, 也类似的有一个独立的xml.

```xml
<account-authenticator 
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:accountType="org.weyoung.notebook"
    android:icon="@drawable/ic_launcher" 
    android:smallIcon="@drawable/ic_launcher" 
    android:label="@string/app_name"/>
```

这里声明的`label`与`icon`都会在系统设置Account里面看到, 如[上一节](http://talentprince.github.io/blog/2015/02/04/android-gou-jian-zhang-hao-yu-tong-bu-fu-wu-part-two/)最后面图片的上部分所示.

----------------------

####使用账号服务

构建好了自己的Service跟账号认证Adapter, 下面就剩如何使用了. 使用大致分为两种, 一种是直接申请Token, 一种是添加账号. 当然如果没申请到, 结果跟后面一种类似.

所有的操作都来自于`AccountManager`,申请Token主要是通过<a href="http://developer.android.com/reference/android/accounts/AccountManager.html#getAuthToken(android.accounts.Account,%20java.lang.String,%20android.os.Bundle,%20android.app.Activity,%20android.accounts.AccountManagerCallback%3Candroid.os.Bundle%3E,%20android.os.Handler)">AccountManager#getAuthToken </a>系列方法. 这里使用了适合第三方应用使用的<a href="http://developer.android.com/reference/android/accounts/AccountManager.html#getAuthTokenByFeatures(java.lang.String,%20java.lang.String,%20java.lang.String%5B%5D,%20android.app.Activity,%20android.os.Bundle,%20android.os.Bundle,%20android.accounts.AccountManagerCallback%3Candroid.os.Bundle%3E,%20android.os.Handler)">AccountManager#getAuthTokenByFeatures</a>, 通过账号类型与Token类型来申请.

添加账号则通过<a href="http://developer.android.com/reference/android/accounts/AccountManager.html#addAccount(java.lang.String,%20java.lang.String,%20java.lang.String%5B%5D,%20android.os.Bundle,%20android.app.Activity,%20android.accounts.AccountManagerCallback%3Candroid.os.Bundle%3E,%20android.os.Handler)">AccountManager#addAccount</a>, 添加指定类型的账号.

当然先通过[AccountManager#getAccountsByType](http://developer.android.com/reference/android/accounts/AccountManager.html#getAccountsByType\(java.lang.String\))查看是否存在账号, 决定是否是添加, 还是获取

当然这些操作都需要一定的权限声明, 如下所示

```xml
    <!-- getAuthTokenByFeatures -->
    <uses-permission android:name="android.permission.USE_CREDENTIALS" />
    <uses-permission android:name="android.permission.GET_ACCOUNTS" />
    <!-- getAccountsByType -->
    <uses-permission android:name="android.permission.MANAGE_ACCOUNTS" />
```

---------------

本文通过三次的介绍, 应该可以为想构建自己的账号同步系统的同学指明了道路, 所有代码中间可能做过稍许节选, 添加了中文的注释, 并有一定的伪代码. 整个程序的代码可以通过[这里](https://github.com/talentprince/Notebook)来获得, 进一步了解整个功能的实现.

--------------

春节将至, 祝快乐

-------------

####Reference:

* http://developer.android.com/reference/android/accounts/AccountManager.html
* http://developer.android.com/training/sync-adapters/creating-authenticator.html