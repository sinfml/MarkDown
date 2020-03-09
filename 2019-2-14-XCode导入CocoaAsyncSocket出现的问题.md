---
layout:     post
title:      XCode导入CocoaAsyncSocket出现的问题
subtitle:   导入framework时产生的种种问题
date:       2019-2-14
author:     sinfml
catalog: true
tags:
    - iOS
    - XCode
---

### Q1

导入https://github.com/robbiehanson/CocoaAsyncSocket release中的framework

单纯copy到目录中并导入,出现Library not found : ..../CocoaAsyncSocket的问题!

### fix1

第一次解决是将Linked Frameworks and Libraries的对应项从Required 改成 Optional,

但是这不能从根本解决问题,但去实现代码时,怎么都连接不上socket

### Q2

引出怎么都连接不上Socket的问题。

### fix2

Optional选项是有需导入,但并不能从根本解决问题.

最后是从Build Phaes中,点击左上角 + 号,点击New Copy Files Phase导入Framework解决!
