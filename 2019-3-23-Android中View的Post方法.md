---
layout:     post
title:      2019-3-23-Android中View的Post方法
subtitle:   
date:       2019-3-23
author:     sinfml
catalog: true
tags:
    - Android
---

### 前提

在做一个需求，就是某个功能第一次使用时，需要弹出一个PopupWindow出来，一开始是这样做的；

### 代码

```Android

protect void onCreate(Bundle savedInstanceState){
        super.onCreate();
        ...

        PopupWindow.show();//显示popupwindow
        ...
}

````

但是,运行的时候就会报错;

```
Unable to add window – token null is not valid; is your activity running?

```

想了下是否Activity未创建完成,所以再移到onResume()里面执行;
但是还是报同样的错误;

后来尝试了这样:

```Android
protect void onCreate(Bundle savedInstanceState){
        super.onCreate();
        ...

        new Handler().postDelay(new Runnable(){
                public void run(){
                        PopupWindow.show();//显示popupwindow
                }
        } , 300);
        ...
}
```

可以了,确定是因为Activity未绘制完成,去调用popupWindow导致闪退;
但是这样写好像有点蠢.
后来查了下,可以这样写:


```Android
protect void onCreate(Bundle savedInstanceState){
        super.onCreate();
        ...

        view.post(new Runnable(){ //view可以当前页面的某个view
                public void run(){
                       PopupWindow.show();//显示popupwindow 
                }
        });
        ...
}
```

因此看了下view中的Post方法:

```
    public boolean post(Runnable action) {
        final AttachInfo attachInfo = mAttachInfo;
        if (attachInfo != null) {
            return attachInfo.mHandler.post(action);
        }

        // Postpone the runnable until we know on which thread it needs to run.
        // Assume that the runnable will be successfully placed after attach.
        getRunQueue().post(action);
        return true;
    }

```
看了下,意思是,如果view已经attach到当前Window中了,就直接调用UI线程的handler直接post过去;
如果未attach的话,就将Runnable放入Runqueue中;


再看这个getRunQueue()中:注释已经很明显了,Assume that the runnable will be successfully placed after attach
再看post跳转后的这个类HandlerActionQueue 中的executeActions方法,该方法就是执行队列下所有任务;
```
public void executeActions(Handler handler) {
        synchronized (this) {
            final HandlerAction[] actions = mActions;
            for (int i = 0, count = mCount; i < count; i++) {
                final HandlerAction handlerAction = actions[i];
                handler.postDelayed(handlerAction.action, handlerAction.delay);
            }

            mActions = null;
            mCount = 0;
        }
    }

```


再去看哪里调用了这个方法,最后又回到了View中的void dispatchAttachedToWindow(AttachInfo info, int visibility)这个方法中;
```
void dispatchAttachedToWindow(AttachInfo info, int visibility){
        ...
        // Transfer all pending runnables.
        if (mRunQueue != null) {
            mRunQueue.executeActions(info.mHandler);
            mRunQueue = null;
        }
        ...
}

```

方法名很明显了,当View attach到Window后,执行这个方法;
所以这个方法相当于界面已经完全绘制完成后,再调用;
所以有种用法是在Activity中获取某个View的实际尺寸,不用这个方法可能会获取长宽都是0的那种情况。



### end







