---
layout: post
title: "Android view touch 事件分发机制之我的理解"
description: ""
category: 
tags: [android, view touch event dispatch]
---

2015 年 9 月份的时候把 view 的事件分发机制研究得觉得自己掌握得很好了，没有记录到笔记上，结果最近要修改第三方自定义控件时，又觉得有点迷糊了，索性重新再学习了一遍，画了一张图来加深自己的理解：

![android_view_touch_event]({{site.img_url}}/android_view_touch_event.png)

其它一些要点 (下文中 View 和 ViewGroup 是分开描述的，View 就是 View，并不指代 ViewGroup)：

1. View/ViewGroup 的 dispatchTouchEvent 在 `ACTION_DOWN` 时返回 true 表示消费了 touch 事件，返回 false 表示不消费 touch 事件。是否消费 touch 事件只取决于 `ACTION_DOWN` 时返回值。
2. Android view 的事件分发机制，是一个由顶层 Activity 开始，中间经历多个 ViewGroup，传递到最底层的 View (或 ViewGroup)，再由底层传回顶层的过程。
3. 当传递到 View 时，说明已经达到一条事件传递路径的最底层，然后开始回溯。View 只可能处于一条传递路径的最底层，且最多只有一个。它不可能处于中间，因于它没有子 View/ViewGroup。处于中间的必须都是 ViewGroup，可能有多个。 
4. 当执行到 onTouchEvent 时，说明事件处于回溯的过程中，事件由底层从顶层传递的过程中，不会再往底层分发。
5. Activity/ViewGroup/View 的 dispatchTouchEvent 方法功能各不相同。只有 ViewGroup 有 onInterceptTouchEvent 和遍历子 View/ViewGroup 的方法；ViewGroup 在拦截 touch 事件时或者子 View/ViewGroup 都没有消费 touch 事件 (即它们的 dispatchTouchEvent 都返回了 false) 的情况下，就会把自己当成一个普通的 View，调用父类 View 的 dispatchTouchEvent；View 的 dispatchTouchEvent 功能很简单，如果有 touch listener 的话，优先执行 listener.onTouch，如果返回 false，则再执行 onTouchEvent；Activity 没有 touch listener。
6. 一次完整的 touch 事件，开始于 `ACTION_DOWN`，终止于 `ACTION_UP`。只有/只要(除非被上层 ViewGroup 拦截) 消费了 `ACTION_DOWN` 事件，才/就 会收到后续事件，否则不会收到除了 `ACTION_DOWN` 外的其它事件。
7. ViewGroup 的遍历子 View/ViewGroup 的行为在一次完整的 touch 事件过程中，只会执行一次，就是当事件为 `ACTION_DOWN` 时。事件 `ACTION_DWON` 是一个完整 touch 事件过程的开始，此时，需要通过遍历来定位一条路径，当路径确认之后，就不再需要遍历，后续的事件就沿着这条确定的路径传递。
8. ViewGroup 的 onInterceptTouchEvent，在传递的路径确定以后 (即之后传递的事件为 `ACTION_DWON` 的后续事件)，一般情况下，这个方法在每次事件到来时都会调用，但有一个例外，当这个 ViewGroup 处于传递路径的最底层时，此时它就相当于一个普通的消费了 touch 事件的普通 View，因此会直接跳过 onInterceptTouchEvent，直接执行父类 View.dispatchTouchEvent (listener.onTouch 或者 onTouchEvent)。
9. 在传递路径确定以后，当路径中的 ViewGroup 一旦决定拦截目标子 View/ViewGroup 的某个事件后 (如 ScrollView 不会拦截子 View/ViewGroup 的 `ACTION_DOWN` 事件，但会拦截子 View/ViewGroup 的 `ACTION_MOVE` 事件)，就会给目标子 View/ViewGroup 发送 `ACTION_CANCEL` 的事件，并把目标子 View/ViewGroup 重置为 null，等同于此 ViewGroup 成为新的传递路径的最底层，因此，正如 8 所说，后续事件到来时，将不会再进入 onInterceptTouchEvent，而是直接调用父类 View.dispatchTouchEvent。

ViewGroup 在 onInterceptTouchEvent 返回 true 后将目标子 View/ViewGroup 置 null 的代码：

Android ViewGroup 早期源码 (不知哪个版本)，摘自[郭霖博客：Android事件分发机制完全解析，带你从源码的角度彻底理解(下)](http://blog.csdn.net/guolin_blog/article/details/9153747)

``` java
if (!disallowIntercept && onInterceptTouchEvent(ev)) {  
    final float xc = scrolledXFloat - (float) target.mLeft;  
    final float yc = scrolledYFloat - (float) target.mTop;  
    mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;  
    ev.setAction(MotionEvent.ACTION_CANCEL);  
    ev.setLocation(xc, yc);  
    // 发送 ACTION_CANCEL 事件给 mMotionTarget
    if (!target.dispatchTouchEvent(ev)) {  
    }  
    // 将 mMotionTarget 置 null，从而让自己成为传递路径的最底层，截断事件继续往下传递
    mMotionTarget = null;  
    return true;
}
```

Android ViewGroup 最新源码 (API 23)

``` java
// Dispatch to touch targets, excluding the new touch target if we already
// dispatched to it.  Cancel touch targets if necessary.
TouchTarget predecessor = null;
TouchTarget target = mFirstTouchTarget;
while (target != null) {
    final TouchTarget next = target.next;
    if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
        handled = true;
    } else {
        final boolean cancelChild = resetCancelNextUpFlag(target.child)
                || intercepted;
        // 如果 intercepted 为 true，给 target 发送 ACTION_CANCEL 事件
        if (dispatchTransformedTouchEvent(ev, cancelChild,
                target.child, target.pointerIdBits)) {
            handled = true;
        }
        if (cancelChild) {
            if (predecessor == null) {
                // 如果 intercepted 为 true，将 target 置 null (next 值此时一般为 null)
                // 从而截断事件往下传递
                mFirstTouchTarget = next;
            } else {
                predecessor.next = next;
            }
            target.recycle();
            target = next;
            continue;
        }
    }
    predecessor = target;
    target = next;
}
```

#### 参考资料：
网上这方面的文章其实不少，但真正说清楚的没几篇，有些洋洋洒洒，其实根本没有说到点子上去，有些则是误人子弟，比如说什么 "当 onTouchEvent 返回 false 时事件就会继续传递到子 View" 之类的 (为什么是错误的见上面第四条)。当然我这篇也不见得都正确，所以事先声明一下，免得误导了你。而且我这篇写得一点都不详细，只能算是写给自己看的一篇粗略笔记。

Trinea 的这篇文章我觉得是讲得最清楚的，其实算是翻译。

1. [Trinea 博客文章](http://www.trinea.cn/android/touch-event-delivery-mechanism/)
2. [英文原文 PDF](http://wugengxin.cn/download/pdf/android/PRE_andevcon_mastering-the-android-touch-system.pdf)
3. [网友 Ocean 翻译的视频](http://v.youku.com/v_show/id_XODQ1MjI2MDQ0.html) (这个视频超赞，推荐)

郭霖也写了两篇博客讲解 View touch 事件传递机制，但我觉得对于初学者来说不是很容易理解，不过源码分析那部分还是不错的。

1. [Android事件分发机制完全解析，带你从源码的角度彻底理解(上)](http://blog.csdn.net/guolin_blog/article/details/9097463)
2. [Android事件分发机制完全解析，带你从源码的角度彻底理解(下)](http://blog.csdn.net/guolin_blog/article/details/9153747)

另外最近买了任玉刚的 《Android 开发艺术探索》，书中这部分内容写得也很详细，推荐。





















