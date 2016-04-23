---
layout: post
title: "Android Handler 中 sendMessage/sendMessageDelayed 与 post/postDelayed 的区别"
description: ""
category: 
tags: [Android,Handler,sendMessage,sendMessageDelayed,post,postDelayed]
---

和 Windows 界面编程中 SendMessage 表示阻塞式的发消息，PostMessage 表示非阻塞的发消息不一样，Android 中 send, post 都是非阻塞式。 sendMessage/sendMessageDelayed 与 post/postDelayed 这四者在内部都是用 sendMessageDelayed 函数实现的，本质上没有什么区别。

使用 post/postDelayed 的好处是代码的简洁，它们的参数是 Runnable 对象，而不是一个 message 对象（当然，在 Handler 内部被自动包装成了一个 message）。

像下面这段代码：

    public void onRefresh() {
        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                swipeLayout.setRefreshing(false);
            }
        }, 5000);
    }

如果改用 sendMessageDelayed 来实现，需要把代码分布在至少三个地方。  
其一，定义一个 message_id 常量

    private final int MSG_ID = 0x1234;

其二，定义 Handler 变量并重写 handleMessage 方法

    private Handler handler = new Handler () {
        @Override
        public void handleMessage(Message msg) {
            if (msg.what == MSG_ID) {
                 swipeLayout.setRefreshing(false);
            }
        }
    }

其三，发消息

    public void onRefresh() {
        Message msg = Message.obtain();
        msg.what = MSG_ID;
        handler.sendMessageDelayed(msg, 5000);
    }

如果发送的消息里要带数据，那还是用 sendMessage/sendMessageDelayed 方便一些，如果用 post/postDelayed 实现的话，需要自己先定义一个实现 Runnable 接口的类，相比之下代码也简洁不到哪去。

如下所示：

    public class TestRun implements Runnable {
        private int test_value;

        public TestRun(int v) {
            test_value = v;
        }

        @Override
        public void run() {
            System.out.println(test_value);
        }
    }

    new Handler().post(new TestRun(5));

因此，在我看来，Android 的 Handler 类实际综合了两种设计思想，一种是命令者模式，一种是通过 message_id 进行消息映射。

前者类似 chrome base 库的线程模型，后者你可以说类似 MFC 的消息机制（但 MFC 没有类似 handleMessage 的函数），如果看过 libjingle 代码的话，就会发现和 libjingle 是很相似的，libjingle 有一个 MessageHandler 类，它有一个 OnMessage 方法，和 Handler 类的 handleMessage 简直是一样一样的。

我在公司做项目的时候，早期同时使用了 chrome base 库和 libjingle 库，我就是通过把 chrome base 库的 Task （即相当于上面的 Runnable）包装成了 libjingle 中的 message，以使两者的线程模型进行兼容。
