---
layout:     post
title:      2019-3-20-启动Activity不使用序列化传递对象
subtitle:   防止因对象数据量过多,序列化失败导致启动Activity报错
date:       2019-3-20
author:     sinfml
catalog: true
tags:
    - Android
---

### 问题

因对象内容过大,在构建内容时,可能忽略某些参数,导致在Activity传递时,会产生因数据会为null导致序列化失败,从而闪退的问题;

因此尝试用enum来保存需要传递的数据,再在目标类中,实现静态方法启动目标Activity

### 代码


```Android

private Object object; //类中变量

private enum DataHolder{
        INSTANCE;

        private Object object;

        public boolean hasData(){
                return INSTANCE.object != null;
        }

        public static void setData(final Object object){
                INSTANCE.object = object;
        }

        public static Object getData(){
                final Object object = INSTANCE.object.clone(); //clone 最好实现一下
                INSTANCE.object = null;
                return object;
        }
}

public static void startActivity(Context srcContext , Object object){
        if (srcContext == null || object == null){
                return;
        }
        Intent intent = new Intent(srcContext , Activity.class);
        DataHolder.setData(object);
        srcContext.startActivity(intent);
}

protected void onCreate(){
        ....
        //在这里获取数据
        if (DataHolder.hasData()){
                this.object = DataHolder.getData();
        }else{
                finish();
        }
}


```