---
layout: post
title: "省略向 Adapter 传递 Context"
description: ""
category: 
tags: [Android, Adapter, Context]
---

我在 Android 开发中很喜欢用 IconFont 来显示图标，用到的库是 [android-iconify](https://github.com/JoanZapata/android-iconify)。这个库提供了一个 demo app，用来展示所有的 icon，但是没有搜索功能，这样想找一个特定主题的 icon 就很不方便。于是上个周末就花了点时间，给这个 demo app 加上了搜索功能，以及点击某个 icon 后显示相似的 icon，并给作者提了 [PR](https://github.com/JoanZapata/android-iconify/pull/166)。

在增加这个功能以及阅读源码的过程中，受到启发，不需要从 Activity/Fragment 为 Adapter 传递一个 Context，直接从 `View.getContext()` 得到 Context 即可(而 Adapter 中 View 无处不在)。示例代码如下。

```java
@Override
public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
    View view = LayoutInflater
            // ViewGroup.getContext()
            .from(parent.getContext())                               
            .inflate(R.layout.item_icon, parent, false);

    view.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            String iconName = (String) v.getTag();
            // View.getContext()
            SimilarIconsActivity.launch(v.getContext(), iconName);   
        }
    });

    return new ViewHolder(view);
}

@Override
public void onBindViewHolder(ViewHolder viewHolder, int position) {
    Icon icon = icons.get(position);
    viewHolder.icon.setText("{" + icon.key() + "}");
    viewHolder.name.setText(icon.key());

    viewHolder.itemView.setTag(icon.key());
}
```

同理，如果需要在 ViewHolder 中使用到 Context，比如需要动态地获取文本、颜色的值，也不需要向 ViewHolder 中传递 Context 了，直接用 `itemView.getContext()`。

PS: 不过在正式的项目中，我一般不会直接在 Adapter 中直接处理点击事件，而是回调给上层的 Fragment/Activity 处理。
