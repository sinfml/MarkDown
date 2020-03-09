---
layout:     post
title:      Android TextView走马灯
subtitle:   1.Android走马灯实现; 2.走马灯滚动抽搐
date:       2020-3-5
author:     sinfml
catalog: true
tags:
    - Android
---

### 问题

TextView实现走马灯效果;
部分情况下走马灯会产生滚动抽搐的效果.

### 代码

主要三个字段:
设置为单行
singleLine="true"
设置为滚动 (另外可以设置为end之类的,效果就是超长后,字符串后面变为省略号)
ellipsize="marquee"
设置滚动次数
marqueeRepeatLimit="3"

```Xml
      <TextView
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:gravity="center"
                android:singleLine="true"
                android:ellipsize="marquee"
                android:marqueeRepeatLimit="3"
                android:textColor="@color/app_bg"
                android:textSize="@dimen/sp_17"
                android:textStyle="normal" />
```
```Android
        textView.setSelected(true); //主动调用即产生走马灯效果; false为取消滚动
```

另外,若出现滚动抽搐的情况,使用RelativeLayout包裹住TextView,并将TextView的width以及height设置为match_parent

```Xml
    <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="@dimen/dp_30"
            android:layout_marginTop="@dimen/dp_31">

            <TextView
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:gravity="center"
                android:singleLine="true"
                android:ellipsize="marquee"
                android:marqueeRepeatLimit="3"
                android:textColor="@color/app_bg"
                android:textSize="@dimen/sp_17"
                android:textStyle="normal" />

        </RelativeLayout>

```