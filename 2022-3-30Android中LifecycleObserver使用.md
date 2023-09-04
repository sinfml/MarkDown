---
layout:     post
title:      Android中ContentObserver使用
subtitle:   使用ContentObserver监听属性变化
date:       2022-3-30
author:     sinfml
catalog: true
tags:
    - Android
---

## 前言

监听Android系统设置属性变化，从而作出响应执行相对应的操作

## ContentObserver

很多时候需要监听数据库数据发生变化后，再去响应对应的操作去使用数据库的新内容，在这种情况下，重新query并不是一个好的选择。因此，可以使用ContentObserver捕捉特定Uri引起的数据库变化，继而做一些响应的操作。

#### 1、首先创建一个继承ContentObserver的类，并且实现onChange（boolean selfChange)方法。

```Java
public class MyObserver extends ContentObserver {

    public MyObserver(Handler handler) {
        super(handler);
    }

    @Override
    public void onChange(boolean selfChange) {
        //do sth.
    }
}

```

#### 2、注册ContentObserver

使用registerContentObserver(Uri uri, boolean notifyForDescendants, ContentObserver contentObserver)注册。Uri代表需要监听的Uri，notifyForDescendatns代表是否需要模糊监听，false为精确监听，当设置为true的时候，可以理解为sql语句中的Like xx%；contentObserver即是我们的Oberser实例。


```Java
    //例如监听adb是否打开
    Handler handler = new Handler();
    MyObserver myObserver = new MyObserver(handler);

    Uri uri = Settings.System.getUriFor(Settings.System.ADB_ENABLED);
    context.getContentResolver.registerContentObserver(uri, false, myObserver);

```

#### 3、取消注册

```Java
    context.getContentResolver.unregisterContentObserver(myObserver);
```

#### 4、处理onChange()

当监听成功并且对应Uri发生变化时，onChange会被触发。

```Java

@Override
public void onChange(boolean selfChange) {
    int isAdbEnabled = Settings.System.getInt(context.getContentResolver(), Settings.System.ADB_ENABLED);
    //handle it
}

```


#### 5、当出现监听无效的情况

这个需要查看源码，确认更新数值的时候是否有调用getContentResolver().notifychange(uri, null)方法，若无调用，是无法监听到变化的。