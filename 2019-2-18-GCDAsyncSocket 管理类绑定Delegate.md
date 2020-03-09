---
layout:     post
title:      2019-2-18-GCDAsyncSocket 管理类绑定Delegate
subtitle:   出现绑定GCDAsyncSocket的Delegate无效
date:       2019-2-18
author:     sinfml
catalog: true
tags:
    - iOS
    - XCode
---

### 问题

上次导入GCDAsyncSocket之后,即使已经能进行他通讯了

但还是不能产生连接上的回调

### fix

后来发现是因为delegate会被系统释放掉,那么只能将管理类改成singleton单例解决

### 代码
```object-c

#import<CocoaAsyncSocket/CocoaAsyncSocket.h>

#define PORT 4444

enum SocketState{
        STATE_NONE       = 1 << 0,
        STATE_LISTEN     = 1 << 1,
        STATE_CONNECTING = 1 << 2,
        STATE_CONNECTED  = 1 << 3
}

@interface ......()<GCDAsyncSocketDelegate>{
        int mState;
        GCDASyncSocket *socket;
}
@end

@implementation .......
singletonM(SocketServer); // 单例申请!!

-(id)init{
        self = [super init];
        if(self){
                mState = STATE_NONE; //初始化状态
        }
        return self;
}

#pragma mark public method
-(void)connect:(NSString *)host onPort:(uint16_t)port{
        //连接方法,传入ip地址已经端口号即可
        if(socket == null){
                socket = [GCDAsyncSocket alloc] initWithDelegate:self delegateQueue:dispatch_get_main_queue()];
        }
        mState = STATE_CONNECTING; //开始连接状态 
        NSError *error = nil;
        if(![socket connectToHost:host onPort:port error:&error]){
                //连接失败!
                NSLog(@"conenct to %@ erro ! %@"  , host , error);
        }else{
                //连接成功!
        }
}

-(void)write:(NSString *)data{
        //通过socket写入数据
        if(mState == STATE_CONNECTED){
                [socket writeData:[data dataUsingEncoding:NSUTF8StringEncoding] withTimeout:-1 tag:0];
        }
}

-(void)cancel{
        //断开连接
        if(mState == STATE_CONNECTED){
                [socket disconnect];
        }
}

#pragma mark Socket delegate
-(void)socket:(GCDAsyncSocket *)sock didConnectToHost:(NSString *)host port:(uint16_t)port{
        //连接成功的回调,将连接状态设备Connected
        mState = STATE_CONNECTED;
}

```

....//同样将剩余的协议补充



