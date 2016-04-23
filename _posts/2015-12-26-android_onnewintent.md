---
layout: post
title: "Android activity onNewIntent 调用时机"
description: ""
category: 
tags: [android, activity onnewintent, lifecycle]
---

最近用到 onNewIntent，对它的调用时机一直存疑，google 了一下，搜到了几个结果都是说 onNewIntent() --> onRestart() --> onStart()，而 [android 开发者官网](http://developer.android.com/intl/zh-cn/reference/android/app/Activity.html#onNewIntent(android.content.Intent)) 上只提到 onNewIntent() --> onResume()。在我的例子中，调用 onNewIntent 后并没有进入到 onRestart() 和 onStart() 中。于是在 BaseActivity 中重写了所有生命周期函数，加入了 Log。代码如下所示：

```java
////////////////////////////////////////////////
// TAG

private String TAG;

protected String getTAG() {
    return TAG;
}

protected void setTAG() {
    this.TAG = this.getClass().getSimpleName();
}

////////////////////////////////////////////////
// lifecycle

@Override
protected void onCreate(Bundle savedInstanceState) {
    setTAG();
    LogUtil.d(getTAG(), "--onCreate--");

    super.onCreate(savedInstanceState);
    setContentView(getLayoutResId());
    ButterKnife.bind(this);

    getBundleData();
    initViews();

    initData();

    if (bindEventBus()) {
        EventBus.getDefault().register(this);
    }
}

@Override
protected void onNewIntent(Intent intent) {
    LogUtil.d(getTAG(), "--onNewIntent--");
    super.onNewIntent(intent);
}

@Override
protected void onRestart() {
    LogUtil.d(getTAG(), "--onRestart--");
    super.onRestart();
}

@Override
protected void onStart() {
    LogUtil.d(getTAG(), "--onStart--");
    super.onStart();
}

@Override
protected void onResume() {
    LogUtil.d(getTAG(), "--onResume--");
    super.onResume();
}

@Override
protected void onPause() {
    LogUtil.d(getTAG(), "--onPause--");
    super.onPause();
}

@Override
protected void onStop() {
    LogUtil.d(getTAG(), "--onStop--");
    super.onStop();
}

@Override
protected void onDestroy() {
    LogUtil.d(getTAG(), "--onDestroy--");
    super.onDestroy();
    if (bindEventBus()) {
        EventBus.getDefault().unregister(this);
    }
}

```

调试后整理 Activity 的 onNewIntent 调用时机如下图所示。

![git_update_author_1]({{site.img_url}}/activity_onnewintent.png)

当 activity (假设为 A) 的 launchMode 为 singleTop 且 A 的实例已经在 task 栈顶，或者 launchMode 为 singleTask 且 A 的实例已在 task 栈里 (无论是栈顶还是栈中)，再次启动 activity A 时，便不会调用 onCreate() 去产生新的实例，而是调用 onNewIntent() 并重用 task 栈里的 A 实例。

如果 A 在栈顶，那么调用顺序依次是 A.onPause() --> A.onNewIntent() --> A.onResume()。A 的 launchMode 可以是 singleTop 或者是 singlTask。[android 开发者官网](http://developer.android.com/intl/zh-cn/reference/android/app/Activity.html#onNewIntent(android.content.Intent)) 上描述的是这种情况。

如果 A 不在栈顶，此时它处于 A.onStop() 状态，当再次启动时，调用顺序依次是 [A.onStop()] --> A.onNewIntent() --> A.onRestart() --> A.onStart() --> A.onResume()。A 的 launchMode 只能是 singleTask。google 到的其它大多文章描述的是这种情况。

--------------------

另外，网上的文章在谈及 activity 的生命周期时，往往只说明单个 activity 的生命周期，而不说明从一个 activity 进入到另一个 activity 时，或者从一个 activity 返回到上一个 activity 时这些函数的调用顺序。现整理如下图所示：

![git_update_author_1]({{site.img_url}}/android_activity_lifecycle.png)

可见原则是优先把目标 activity 尽快展示出来，等目标 activity 展现出来后，再在后台执行自身的 onStop，或者以及 onDestroy。而并不是先执行完自己的 onStop/onDestroy 再去执行目标 activity 的 onCreate/onRestart。
