---
layout:     post
title:      Android无障碍功能无法访问微信的xml的解决方案
subtitle:   无障碍dumpXML
date:       2025-7-15
author:     sinfml
catalog: true
tags:
    - Android 
---

## 前提

微信有风险控制，导致使用无障碍功能无法获取某些界面的xml布局信息，因此需要伪造无障碍类的包名以及类名绕过微信的风控机制。

## 无障碍服务按照如下配置即可（重点是android:name属性）

```xml
<service
    android:name="com.google.android.accessibility.selecttospeak.SelectToSpeakService"
    android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE"
    android:exported="true"
    android:enabled="true">
    <intent-filter>
        <action android:name="android.accessibilityservice.AccessibilityService"/>
    </intent-filter>
    <meta-data
        android:name="android.accessibilityservice"
        android:resource="@xml/accessibility_service_config"/>
</service>

```

类要放在对应的com.google.android.accessibility.selecttospeak包名下。