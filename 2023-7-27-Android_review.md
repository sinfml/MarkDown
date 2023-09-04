---
layout:     post
title:      Android_review
subtitle:   知识点
date:       2023-7-27
author:     sinfml
catalog: true
tags:
    - Android
    - Java
    - 音频
---
---
# Java
## JVM内存结构
- 堆 （对象在堆、常量在栈，常量在对象中就在堆） 管道、先进先出、后进后出
- 本地方法栈 Native接口的代码 
- PC寄存器 指令编号
- 本地方法栈 线程私有的常量和引用 瓶子，先进后出，后进先出
- 本地方法区 线程共享的内存区域，虚拟机已经加载的类、常量、静态变量。

---

## 是值传递还是引用传递

值传递。基础类型是传递的值，应用类型传递的是地址，因为地址指向是同一位置，因此修改后还是一致。

---

## GC 回收
- 引用计数
- 可达分析

1. 引用计数算法，当引用数为0时，代表对象可GC。 当如果循环引用，如
```C
obj a = '1'
obj b = '2'
a.id = b
b.id = a
```
则永远无法释放

2. 标记清除，第一阶段标记不可达的内容，第二阶段清除掉标记的。但会造成内存碎片。

3. 标记整理清除，在标记清除中间插入整理，将标记的区域归一化，清除直接清除掉这一片区域，但移动工作会耗费资源。

4. 分代算法，分年轻代、中生代、年老代，频繁GC的内容在年轻代释放，但年轻代满了就移动到中生代（2个），如果2个中生代都满了，就移动到老年代就做标记整理清除算法，再GC。

---

## final finally finalize
- final 声明，不可修改
- finally 用于try-catch，最终会执行
- finalize 方法，对象被清理时会被调用

---

## 抽象类与接口类

- 抽象类是子类的属性集合，讲的是定义一个对象。
- 接口类是相当于对象的行为特征，只能有方法的声明，不能实现。
- 单继承，多实现。

---

## 类锁，对象锁 synchronized(底层是使用了一个类似计数锁Monitor)

就是范围的问题,锁住某个对象，或者某个方法，或者是整个类的实例。

--- 

## wait和sleep的区别

- 都是暂停当前线程，让出CPU
- wait必须在同步块中，而sleep不需要
- wait在超时前，都会持有锁，需要notify，notifyAll（notify只会随机唤醒一个，而notifyAll时所有）才会释放
- sleep不会占用锁，两者都可以被interupt中断。

---

## Java 4个引用

---

## 排序算法

1. 冒泡排序

比较相邻的两个元素，如果第一个比第二个大，就交换两个；针对所有元素操作一遍，除了最后一个。

2. 选择排序

在列表中找到最小的元素，放到首位（交换）；然后切到下一位，再继续步骤，直到最后。

3. 插入排序

将第一个元素看作已排序对象，第二个到最后为未排序对象，抽出一个，往前翻找，找到合适的位置插入（过程中比抽出的大的，可以后移一位）

4. 快速排序

分别从两端出发，以第一个数作为基准，从后面开始找比基准小的，从前面找比他大的。如果两个查找下表还是大>小，则两者交换；再循环知道两个相交，相交位置和基准位置交换。

这时候基准位置在这个队列的中间。 将队列分为2部分，分别对其重复第一步。 循环知道所有排序完毕

```C
void quickSort(int *arr, int begin, int end) {
    if (begin > end) return;
    int tmp = arr[begin];
    int i = begin;
    int j = end;

    while (i != j) {
        while (arr[j] >= tmp && j > i) {
            j--;
        }
        while (arr[i] <= tmp && j > I) {
            i++;
        }
        if (j > i) {
            int t = arr[i];
            arr[i] = arr[j];
            arr[j] = t;
        }
    }
    arr[begin] = arr[i];
    arr[i] = tmp;
    quickSort(arr, begin, i - 1);
    quickSort(arr, i + 1, end);
}
```

---

## 时间复杂度

1. O(1)
```C
int n = 100;
print(n)
```
2. O(logn)
```C
//对数增长
int n = 32;
for (int i = 0; i <= n; i *= 2) {
    print(n)
}
```
3. O(n)
```C
int n = 100;
for (int i = 0; i < n; i++) {
    print(n)
}
```
4. O(nlogn)
```C
int n = 32;
for (int i = 1; i < n; i++) {
    int x = 1;
    while (x < n) {
        x *= 2;
        print(x);
    }
}
```
5. O(n^2)
```C
for (int i = 0; i < 10; i++) {
    for (int j = 0; j < 10; j++) {
        print(i*j);
    }
}
```
---

# Android

## 四大启动模式

### 1.Standard

 默认模式，每启动一个Activity都会启动一个实例；在这种模式下，谁启动了child Activity，这个child Activity就在parent Activity的栈中。

### 2.SingleTop

栈顶复用模式，单例模式，如果栈中存在Activity，且该Activity存在栈顶，则不会重新创建，但会触发onNewIntent方法；但如果不在栈顶，则回重新创建Activity。同一页面，显示不同内容。

### 3.SingleTask

栈内复用模式，单例模式，如果栈中存在Activity，那无论如何都不会启动一个新实例，复用时，在其上方的Activity将全部出栈，并且回触发该Acitivity的onNewIntent方法。 程序入口。

### 4.SingleInstance

单例模式；创建的时候单独一个任务栈，且具有全局唯一性。具有SingleTask的特性，存在即不会重新创建。系统Launcher。 

---

## Activity 生命周期
- onCreate 创建
- onStart 可见
- onResume 状态恢复
- onPause 将要离开
- onStop 不可见
- onDestroy 释放

---

## Fragment 生命周期
- onAttach 与Activity产生关联
- onCreate 创建
- onCreateView 创建将要显示的View
- onActivityCreate 所在Activity启动完成
- onStart 可见,启动Fragment时调用
- onResume 状态恢复
- onPause 准备到后台
- onStop 不可见或停止
- onDestroyView 销毁其中组件
- onDestroy 销毁Fragment
- onDetach 取消跟Activity关联

---

## Handler 
- Handler
- Message
- MessageQueue
- Looper

一个线程只有一个Looper，一个Looper对象中维护一个消息队列MessageQueue，但可以有多个Handler关联这个Looper。


### Handler
用于发送和处理消息，Looper线程不断的从消息队列中获取信息，Handler不断的推送信息到队列中。Handler在工作线程中通过sendMessage向MessageQueue推送信息，message.target就是该Handler。
### Message
消息携带者
### MessageQueue
消息队列，结构和列表类似，每个线程只存在一个消息队列，且根据when决定优先级，越小优先级越高。单向链表，通过next存储下一个Message。
所有信息最终都会插入到这里，插入后会根据一些判断，判断是否需要唤醒阻塞的队列。
### Looper
每一个MessageQueue对应一个Looper，因此每个线程也只存在一个；如果在主线程，还有一个额外的MainLooper。通过Loop方法，调用dispatchMessage进行消息分发。
### 内存泄漏
退出Activity时，Handler还在执行,或者还存在相关的引用。
解决方法，进行弱应用，Activity释放时，中断相关操作。


### Q&A
1. 为什么要通过obtain获取消息
节省系统资源,在消息池中获取空的Message对象。

2. Handler延迟发送信息的原理
最终都会调用到sendMessageAtTime。他不是延迟发送，而是直接发送，再借由MessageQueue的设计对信息进行延迟处理。通过Native方法阻塞线程一定时间，等待消息的执行时间到达后再取出信息处理。

3. 为什么不会卡死主线程


---

## Activity、View、Window的关联

Activiy相当于一个控制器，初始化attach会创建一个PhoneWindow（window的实现），再通过setContenView将布局layout放到dectorView中。

---

## EventBus

遵循事件发布-订阅框架，主要包含Event，订阅者，发布者。

---

## 如何加快启动速度

- 不马上使用的进行异步加载
- 对使用率不高的初始化作懒加载
- 增加开屏页
- 优化布局

---

## 如何对OkHttp进行网络优化

- 连接服用，使用keep-alive
- 减少数据传输的大小，使用压缩gzip
- 根据网络情况下在不同质量的图片

---

## View 绘制流程
View是所有控件的基类，包括ViewGroup（控件组）也是继承自View。

View的位置根据4个属性决定，top、left、bottom、right；是相对于父布局的相对位置；

### 绘制流程
- onMeasure
- onLayout
- onDraw
### onMeasure
用于决定View的高度和宽度
### onLayout
决定视图以及所有子视图的大小以及位置；只有在定义ViewGroup的时候才需要重写。
### onDraw
具体的绘图流程

### Surface
每个Window对应一个Surface，任何View都要画在Surface的Canvas上。Surface将BufferInfo绘制，绘制完成后queueBuffer到BufferQueue；SurfaceFlinger申请数据进行合成，合成后重新释放到Surface。

---

## AIDL
Android接口定义语言，一般用于客户端与服务器通信用的接口语言，一般用于跨进程通讯。
- 可以传输基本类型
- 可以传输例如String，CharSequence
- 实现了Parcelable的类
- List和Map，但必须是包含以上三种类型
- 可嵌套interface aidl作为回调使用

.aidl文件不会负责功能，因此需要以来Service来实现具体方法；使用IBinder来进行使用。
```Android
interface IAidlInterface {
    void test();
}
```
```Andorid
public class OurService extends Service {
    @Override
    public IBinder onBind(Intent intent) {
        return new OurBinder();
    }

    class OurBinder extends IAidlInterface.Stub {
        @Override
        void test() {
            //handle it
        }
    }
}
```
将.aidl文件拷贝到需要使用的工程中，并通过ServiceConnection绑定该服务使用
```Android
private ServiceConnection connection = new ServiceConnection() {

    @Override
    public void onServiceConnected(ComonentName name, IBinder service) {
        IAidlInterface binderInterface = IAidlInterface.Stub.asInterface(service);
        binderInterface.test(); //调用方法
    }

};
```

--- 
## 内存泄漏以及处理
如果对象已经不需要使用，本来该回收的时候，被别的应用一直持有需要回收对象的引用，会导致无法被GC回收，就会出现内存泄漏；
- 单例
- Handler消息还存在
- 线程未销毁并持有对象
- 资源没关闭

### 解决方式
使用Leakcanary或者Android Studio的Profile

---
## ANR超时时间

- Broadcast为10秒
- UI线程为5秒
- 前台Service为20秒，后台为200秒

---

## MVC、MVP、MVVM
### MVC
C代表界面处理交互、V代表界面中的view显示内容、M代表数据处理以及回调信息给V；
C按下按钮，告诉M层，M层处理完数据回调给V，显示内容。
### MVP
基于MVC，将界面交互和内容显示集中在V；M代表数据处理；P层作为M和C的中间件既负责传递V给M的信息，也回调M给V的内容，将2者完全分离；
### MVVM
搭配jetpack中的databinding使用，与MVP类似，VM取代了P；负责VM和V的数据绑定，当VM的数据更新，V层的View会自动显示更新后的内容。

---

## Glide

原理：1.发起请求（生命周期）；2.启动任务（指定到Target）；3.解码图片；4.缓存；5.初始化和配置；6.加载选项

### 缓存机制
三级缓存
分为内存缓存、磁盘缓存和来源缓存。内存缓存中又分为Lru缓存和弱引用缓存。
首先从弱引用缓存取，没有则去Lru缓存中取，发现则添加到弱引用缓存中。如果都没有就开始从磁盘缓存中寻找。
磁盘缓存就是通过对应的key去DisckCache中寻找。
- LruCache通过维护一个LinkedHashMap来存储数据；其中的机制就是最近最少使用原则，如果缓存的数据超过默认大小，则会把最久使用的图片移除。（栈底淘汰机制）

### 生命周期管理
主要对两类不同的Context进行不同的处理。
ApplicationContext就是整个应用声明周期绑定；其余Context就是在Activity中创建无布局的fragment，绑定声明周期。

### 内存抖动
使用BitmapPool，新建图片时，会从BitmapPool里面找有没有对应大小的Bitmap，有则直接使用，没有才会申请新的Bitmap；回收的时候，回提交给BitmapPool，供下次使用。

### 自定义对象
重写GlideModule，通过重写registerComponents
```Android
    @Override
    public void registerComponents(Context context, Glide glide) {
        glide.register(Object.class, InputStream.class, new CoverLoader.Factory());
    }
```
新建ModelLoader<Object, InputStream>，Object代表自定义对象，InputStream代表Object转化后的图片数据。实现DataFetcher，用于解析传入的Object。

---

## OkHttp

优势
- 支持HTTP/2
- 连接池
- 响应缓存
- 支持同步异步
- 其他

拦截器，每一个拦截器构成一个拦截器链。当执行操作时，会触发拦截器链的Process，调用拦截器的intercept，因为每个拦截器有拦截器链的引用，在拦截器里面也会调用process，并且下标加一。直到最后

---

## RxJava

主要有4个模块组成，发布者、订阅者、订阅

RxJava中，onError和onComplete有且只有一个，都会使事件链中断。

操作符：
1. from 从数组逐个发出
2. just 单个或多个发出，多个会转换成from
3. interval 固定时间间隔发出
4. range 范围

变换操作符：
1. map 转换成Observable对象，返回的是一个类型
2. flat 跟map类似，但是返回的是Observable<?>类型
3. concatMap 跟flatmap类似，但是是有序的
4. cast 强制转化.class再发出
5. buffer 将一个数组分组发送，例如abcdedfg buffer 3，即abc ded fg

---

## Retrofit

封装了okhttp，请求部分主要还是使用okhttp完成，Retrofit本身对请求方法做了封装。

使用了注解来简化请求，将okhttp的方法抽象成java接口。

原理：通过Builder创建Retrofit对象，其中指定baseUrl，格式转换工厂，okhttp，以及其他属性。然后指定对应实现了接口的Class文件，调用对应接口方法获取数据。

---

## HashMap

Key-Value的链表,不是线程安全的,没有排序的，key和Value都可为null。

---

## Room

jetpack中的数据库

- 使用注解，减少重复和容易出错的样板代码
- 简化了数据迁移

```Android
//声明对象
@Entity
public void Person {

    @PrimaryKey
    public int id;

    @ColumnInfo(name = "first_name")
    public String firstName;
}
```

使用Dao的接口类直接查询
```Android
@Dao
public interface UserDao {
    @Query("SELECT * from user)
    List<Person> getAll();

    @Insert
    void insertAll(Person... persions);

    @Delete
    void delete(Person person);
}

```


---

## 事件分发机制

Activity->ViewGroup->View

dispatchTouchEvent、onInterceptTouchEvent、onTouchEvent

Activity dispatchTouchEvent返回false直接触发Activity的onTouchEvent，否则交给下一级处理

ViewGroup dispatchTouchEvent，执行onInterceptTouchEvent判断是否需要拦截，如果是true，自己执行onTouch，onTouchEvent，performClick，onClick；否则寻找子View的dispatchTouchEvent

View 跟ViewGroup类似，但先判断是否有注册onTouch事件，根据onTouch事件的true/false决定是否消费了该事件；如果没有注册onTouch事件，触发onTouchEvent，performClick的时候判断是否注册了onClickListener，后续的流程一致。

---

## IPC通讯
### 多进程

binder是客户端和服务端通信的媒介，但bindService时，就会返回一个binder，通过binder客户端可以从服务端获取数据，或者执行服务端的方法。这里的服务包括普通的service服务或者AIDL服务。

```xml
<service
    android:name=".service"
    android:enabled="true"
    android.procsss="process_name">  //多进程名字
</service>
```
1. 使用Bundle，四大组建件的通讯，对象可能需要序列化
2. 使用文件共享，多用于单线程读写。
3. 使用Messenger，跟aidl类似，就是Handler和binder的结合
```Android
//服务端
    Messenger messenger = new Messenger(new Handler(){
        @Override
        public void handleMessage(final Message message) {
            //handle it
            ...

            //回复给客户端
            Message reply = Message.obtain();
            Bundle bundle = new Bundle();
            bundle.putString("test", "message");
            reply.setData(bundle);
            //发送
            message.replyTo.send(reply);
        }
    });

    @Override
    public IBinder onBind(final Intent intent) {
        return messenger.getBinder();
    }

//客户端
    //先创建本地的Messenger
    Messenger localMessenger = new Messenger(new Handler() {
        @Override
        public void handleMessage(final Message message) {
            //handle it 
        }
    });

    ...
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        Messenger messenger = new Messenger(service);
        //绑定信息接收
        messenger.replyTo = localMessenger;

        //返回信息
        messenger.send(Message.obtain());
    }
```
4. AIDL

上面有描述

5. ContentProvider

- 主要是定义一个url，暴露给外界，提供query、update、insert、delete等方法。
- 服务端需要在AndroidManifest暴露uri
```xml
<provider
    android:name="provider.name"
    android:authorities="provider.name.uri" //uri
    android:permission="provider.permission" //相应的权限
    android:enabled="true"></provider>
```
相应的实现一些ContentProvider接口提供数据，如同数据库
```Java
public class MyProvider extends ContentProvider {
    ...
    @Override
    public Cursor query(Uri uri, String[] projections, String selection, String[] selectionArgs, String order) {

    }

    ...//以及其他
}
```
- 客户端只需要通过ContentResolver以及暴露的uri进行操作即可。
```Java
    private static final String PROVIDER_URI = "content://provider.name.uri"; //需要content://开头

    ...
    Cursor cursor = getContentResovler().query(PROVIDER_URI, new String[]{...}, null, null, null);
    if (cursor != null) {
        ...
    }
```
6. Socket通讯

各个IPC通讯的优缺点：
- Bundle、文件共享简单易用，但前者需要序列化、后者不是实时;
- AIDL使用线程池，可以一对多通信，但需要处理线程同步；
- Messenger，和AIDL类似，但内部是Handler实现，所以只能串行通讯；
- ContentProvider多用于数据提供，也是一对多；
- Socket 可高度自定义，通过传输字节流 

---

## Scrcpy

大致流程：
1. 获取本机的Rect以及屏幕方向
2. 从SurfaceControl获取displayToken；DisplayToken是一个IBinder对象，返回的是需要投屏的设备ID对应的SurfaceFlinger服务代理。
3. 创建一个MediaCodec，根据本机的Rect配置MediaFormat，并从MediaCodec对象中获取InputSurface对象。
4. 将当前的displayToken交给MediaCodec获取的InputSurface，后续的图像就会传递到这个InputSurface中。
5. 配置完成后，调用MediaCodec Start，进行编解码。
6. 使用dequeue输出到bufferInfo中，就可以对这个bufferInfo作传输处理，这里使用了TCP传输。使用完毕后。调用release释放掉。

---

# 网络

## HTTP和HTTPS的区别

- HTTP采用的是明文传输，没有校验数据完整性，没有验证对方身份
- HTTPS是HTTP+SSL/TLS，增加了SSL证书。服务端将公钥传输给客户端，客户端根据协商的加密等级，客户端利用公钥进行加密一个通信密钥，传送给服务端。服务端再使用私钥对通信密钥进行解密,获取通信密钥;后续的通讯都使用对称加密完成.

- 三次握手
1. 客户端发送SYN请求到服务端
2. 服务端返回SYN和ACK到客户端
3. 客户端发送ACK确认连接

- 四次挥手
1. 客户端发送FIN
2. 服务端收到FIN后,返回一个ACK
3. 服务端发送FIN
4. 客户端收到FIN后,返回一个ACK

---

## WebSocket
### 概念
- WebSocket基于TCP/IP协议，独立于HTTP协议的通信协议。
- 双向通信，可一对一、可多对多。
- WebSocket端口是80,SSL端口是443。

### 优点
- 支持双向通信，实时性强
- 更好的二进制支持
- 较小的控制开销。包头字节少
- 支持拓展，可支持自定义压缩算法

### 建立连接
复用了HTTP的握手通道，具体指客户端通过HTTP请求与WebSocket服务端协商升级协议，完成后，后续的数据交换遵循WebSocket的协议。

### 数据分片
区别与TCP/IP的分包，WebSocket的每条信息可能被且分成多个数据帧，当客户端收到一个数据帧时，会根据FIN的值来判断是否收到信息的最后一个数据帧。

此外，opcode在数据交换的场景下，表示的是数据的类型，0x01表示文本，0x02表示二进制。0x00表示延续帧，代表数据还没接收完毕，0x08表示断开。

### Sec-WebSocket-Key/Accept的作用
主要作用于基础的防护，减少恶意连接，意外连接。

Accept是根据客户端请求首部的Key计算出来
通过Key跟某个字符串传拼接，再通过SHA1（密码散列函数，数据分片为512bit，如果长度不够，增加填充，并另外预留64bit填充原始信息长度）计算出摘要，并转成base64字符串。

---

# 音频

## FFmpeg
### 模块布局
1. libavformat，格式封装，就是分析歌曲格式内容
2. libavcodec，编解码
3. libavutil，工具类
4. libavfilter，滤波器等，音视频效果处理
5. libavdevice， device设备
6. libswscale， scale视频图像缩放
7. libavresample, 重采样
8. libswresample， 音视频重采样

### 常见Context
1. AVFormatContext
```C
1. 创建avformat
ifmt_ctx = avformat_alloc_context();

2. 打开音视频
avformat_open_input(&ifmt_ctx, file_name, 0, 0);

3. 读取frame
while (...) {
    av_read_frame(ifmt_ctxt, &pkt);
}

4. 关闭上下文
avformat_close_input(&ifmt_ctx);
```
2. AVCodecContext
编解码器上下文，用于操作编码器和解码器，实现编码或者解码。
```C
1. 创建avcodec
codec_ctx = avcodec_alloc_context3();

2. 打开编码器
avcodec_open2()

avformat_find_stream_info(ifmt_ctx, NULL);
for (i = 0; i < ifmt_ctx->nb_streams, i++) {
    AVStream *stream = fmt_ctx->streams[i];
    AVCodec *codec;
    if (avcodec_open2(stream->codec, codec, 0) < 0)
    {
    }
}

3. 关闭编码器
avcodec_close()
```

3. AVFilter
主要由FilterGraph以及一个个FilterContext串联组成

Graph

---
abuffer-filter1-filter2-filter3-abuffersink

```C
    AVFormatContext *fmt_ctx = pHandle->fmt_ctx;
    AVCodecContext *dec_ctx = pHandle->c;

    AVFilterGraph *filter_graph;
    AVFilterContext *abuffer_ctx;

    avfilter_register_all();
    filter_graph = avfilter_graph_alloc();

    //配置输入源采样信息
    snprintf(args, sizeof(args),
                "time_base=%d/%d:sample_rate=%d:sample_fmt=%s:channel_layout=0x%" PRIx64,
                time_base.num, time_base.den, dec_ctx->sample_rate,
                av_get_sample_fmt_name(dec_ctx->sample_fmt), dec_ctx->channel_layout);


    const AVFilter *abuffer = avfilter_get_by_name("abuffer");
    err = avfilter_graph_create_filter(&abuffer_ctx, abuffer, "in",
                                        args, NULL, filter_graph);

    //初始化其中一个滤波器
    equalizer = avfilter_get_by_name("equalizer");
    //填充参数并创建Context
    av_dict_set_int(&options_dict, "f", 31, 0);
    av_dict_set_int(&options_dict, "width_type", 3, 0); //QFACTOR
    av_dict_set_int(&options_dict, "filter_type", 1, 0);
    //  av_dict_set_int(&options_dict, "width", WIDTH, 0);
    av_dict_set_int(&options_dict, "g", GAIN, 0);

    err = avfilter_init_dict(equalizer_ctx1, &options_dict);
    av_dict_free(&options_dict);

    //配置输出
    abuffersink = avfilter_get_by_name("abuffersink");
    err = avfilter_graph_create_filter(&abuffersink_ctx, abuffersink, "out", NULL, NULL, filter_graph);

    //一个个Context连接起来
    avfilter_link(abuffer_ctx, 0, equalizer_ctx1, 0);
    avfilter_link(equalizer_ctx1, 0, abuffersink_ctx, 0);
    /* Configure the graph. */
    err = avfilter_graph_config(filter_graph, NULL);

```
---

调用时，把解码完成的数据传入abuffer输入源，再从abuffersink输出读取
```C
    int ret = 0;
    ret = av_buffersrc_add_frame_flags(pHandle->bufferSrc_ctx, pHandle->decoded_frame, AV_BUFFERSRC_FLAG_NO_COPY);
    if (ret < 0)
    {
            av_frame_unref(pHandle->decoded_frame);
            return 0;
    }
    while (ret = av_buffersink_get_frame(pHandle->bufferSink_ctx, pHandle->decoded_frame) > 0)
    {
    }
```

4. libswresample
SwrContext重采样上下文
```C
1. 初始化
swr_ctx = swr_alloc();

2.配置输入输出采样参数
av_opt_set_sample_fmt();

如

av_opt_set_sample_fmt(pHandle->swr_ctx, "in_sample_fmt", pHandle->c->sample_fmt, 0);
/* set options */
av_opt_set_int(pHandle->swr_ctx, "in_channel_layout", my_channel_layout, 0);
av_opt_set_int(pHandle->swr_ctx, "in_sample_rate", pHandle->sampleRate, 0);

av_opt_set_int(pHandle->swr_ctx, "in_channel_count", pHandle->c->channels, 0);
av_opt_set_int(pHandle->swr_ctx, "out_channel_count", 2, 0);

av_opt_set_int(pHandle->swr_ctx, "out_channel_layout", AV_CH_LAYOUT_STEREO, 0);
av_opt_set_sample_fmt(pHandle->swr_ctx, "out_sample_fmt", AV_SAMPLE_FMT_S32, 0);


3. 使用
swr_convert(swr_ctx, out, out_count, in, in_count);
```
---

## DLNA

主要分为几个部分，DMS、DMR、DMP、DMC
- DMS，媒体服务提供者
- DMC，相当于遥控器，将DMS的音频推送到DMR中
- DMP，直接访问DMS内容，并自己播放
- DMR，作为接收端播放

主要是用UPNP服务，分为AVTransport（控制上下曲，播放暂停等），Rendering（控制音量），ContentDirectory（可访问的媒体内容），ConnectionManager（可连接的设备信息）。ControlPoint是一个基本单位，一个设备可以作为一个ControlPoint。

ControlPoint通过discovery服务发现局域网类的其他设备，通过读取设备的描述内容，分析该设备是什么类型的设备，具备什么样的服务。设备控制基于SOAP协议，一种简单XML协议。通过Http请求，通过Http响应。

---

## WebDAV

基于HTTP1.1的通信协议，在GET/POST/HEAD等常用方法中拓展了几个其他方法。使得设备可以直接访问WebServer。

---

## 音频fft

### FFT
快速傅立叶变化，将信号从时域转换到频域。按照采样点获取对应的幅值。通过ifft可转回时域。

---
## H.264

### 压缩技术
1. 帧内预测压缩，解决空域数据和数据冗余
2. 帧间预测压缩，解决时域数据冗余
3. 整数离散余弦变换，将空间上的相关性变为频域上的数据进行量化。
4. CABAC压缩

### 压缩后分为I、P、B帧
1. I帧，关键帧
2. P帧，向前参考帧，参考前面已经处理的帧
3. B帧，双向预测帧。

GOP，图像序列，一个图像序列只有1个I帧。

### 压缩技术
- 划分宏块，默认16X16，计算其像素值，以此类推，计算全部像素值。划分完后，就可以进行帧分组。
- 帧分组。时间冗余，就是相当于1秒抓了30帧，这1秒内的帧关系都比较密切，可能是变化不大的，这就是时间冗余。在这种相似帧中，我们只保留第1帧的数据，其他帧就参考第1帧计算出来。第1帧就是I帧，其他为B/P帧，一个帧组就是GOP。
- 运动估计和补偿，帧分组后，就要计算运动补偿。从缓冲区里面取出两组GOP，进行宏块扫描，当发现一副图片有物体，就在另一副临近位置进行搜查，通过位置差，计算出矢量和距离。
- 帧内预测。高低频区分，对每个宏块进行预测得到图像跟原图进行相减得到残差。对残差再进一步做DCT，去掉相关联数据后再用CABAC压缩（高频配置短码，低频配置长码）

---
## UDP TCP区别
- 有连接和无连接
- 对系统资源占用的多少
- UDP结构较为简单
- 流模式和报文模式
- TCP能保证数据正确性，UDP会丢包
- UDP不保证数据顺序

### TCP 
1. 三次握手。
- A向B提出连接，同步序列号
- B收到后，用带有ACK和SYN响应，再告诉A可以开始连接
- A收到后，再发送一个应答，完成后SYN置0

2. 四次挥手，
- A向B申请中断，FIN置1
- B收到后，发送ACK置1响应
- B发送准备关闭请求，FIN置1
- A收到后，发送ACK响应，双方中断
---


## 字节跳动面试
### 1. Android IO模型

1. BIO
2. NIO
3. select/poll/epoll

### 2. 图片太大如何处理，如果是自己做图片框架，如果实现。

### 3. 判断文件完整性，以及分片后如何确保正确性。

### 4. Retrofit的原理，注解怎么转化为请求，Retrofit可以不可以不用OkHttp。

主要流程： Retrofit通过Java接口以及注解描述网络请求，动态代理的方式生成网络请求request，然后通过client调用响应的网络框架去发起网络请求（默认OkHttp）。返回response后通过converFactory转换成相应的model，再通过calladapter转换成对应的格式。

- 通过Retrofit.create(class)创建出Service Interface实例，从而使得Service中的方法可用。
- 内部使用的是Proxy.newProxyInstance()方法创建对象，这个对象实现了interface的方法。（动态代理方法）

动态代理以及静态代理

- 编译时确定被代理的类时哪一个，即是我们通过AS编写的代码就是静态代理。
```Java
    public interface MyInterface {
        public void say(String str);
    }

    //定义被代理类
    public class A implements MyInterface {
        @Overriade
        public void say(String str) {
            sout(str);
        }
    }

    //定义静态代理类，在原接口上功能增强
    public class B implements MyInterface {
        private MyInterface ii;
        public B(MyInterface interface) {
            this.ii = interface;
        }

        @Override
        public void say(String str) {
            str += " world";
            ii.say(str);
        }
    }

    //当我们不想修改已有的代码，但想更改部分逻辑，采用代理间接访问
    public void test() {
        A a = new A();
        B proxy = new B(a);
        proxy.say("hello");
    }
```

- 动态代理，可以灵活处理，但接口方法比较多的时候，不需要每个代理方法都处理一次。使得功能更加单一以及复用性更强。

其中包含几个重要的类和接口，主要差别在最后的代理实现上，使用实现了接口InvocationHandler的类，传入Object，通过反射调用对应的Method

InvocationHandler
```Java
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args) throw Throwable;
}

/**
 * proxy 需要代理的类的实现
 * Method 反射出来的对应方法
 * args 变量
 * /
```

具体实现，前两部跟静态代理类一样
```Java
//定义一个InvocationHandler，用于处理需要增加的方法。
public class AProxy implements InvocationHandler {

    private Object proxyObj; //被代理的类
    public AProxy(Object obj) {
        this.proxyObj = obj;
    }

    @Override
    public Object invoke(Object object, Method method, Object[] args) throw Throwable {
        //在这里增强实现，可通过method的Name区分不同的方法
        //反射调用Method
        if (method.getName().equals("say")) {
            String tmp = (String) args[0];
            //增强
            tmp += " world";
            method.invoke(object, tmp);
        } else {
            method.invoke(object, args);
        }
    }
}

    //调用
    A a = new A();
    //跟静态类类似
    ProxyA proxyA = new ProxyA(a);
    //生成代理类，其实就是转换成接口
    MyInterface myInterface = Proxy.newProxyInstance(proxyA.getClass().getClassLoader(), proxyA.getClass().getInterfaceClass(), proxyA);
    //就可以直接调用接口的方法
    myInterface.say("test");

```

### 5. OkHttp的原理，怎么判断网络状态（除了网络速度外）。

### 6. Android底层的网络请求还有哪些

### 7. TCP、UDP各生成2000个端口，会有冲突吗

不会有冲突。只有TCP服务端网络编程存在监听端口，UDP不会有。

在数据链路层，通过MAC地址找到对应主机；在网际层，通过IP地址寻找主机或者路由器；通过端口来寻址，来识别同一台计算机中不同的应用程序。
因此，传输层的端口号是用于区分不同应用程序的数据包。
但TCP/UDP是两个不同的传输模块，端口收到信息后，通过IP包头协议号区分是TCP还是UDP。根据这个信息传输到对应的模块中处理。因此两者不会产生端口冲突，

#### 那如果是两个TCP可以用一个端口吗？

可以。但是需要绑定两个IP地址。如果同一个IP地址，端口号不可以复用。

#### 为什么TCP中断后，如果马上再申请绑定端口，会有时候产生Already in use的错误？

因为4次挥手的时候， 对于主动关闭方，会存在2MSL的TIME_WAIT时间。使用so_reuseaddr解决。

#### 那客户端能够使用同一个端口吗？

可以的。其实TCP判断唯一性是需要通过4个变量。源IP地址、源端口、目标IP地址、目标端口。如果服务端是不同的IP，那客户端是可以使用相同的端口。

### 8. Handler，postDelay如何实现的。如果HandlerA post了Message，HandlerB能够remove吗

postDelay最终调用到Handler中的sendMessageAtTime，直接插入到队列中。在native中阻塞，直到到达指定的时间唤醒Looper发送信息。

不能。因为虽然是同一个MessageQueue，但是调用remove的时候，会判断该Message的target即是哪个Handler发送的信息，匹配上才会remove。

### 9. 如何优化启动，除了XML布局优化，转化成Android代码外呢？

1. 线程优化

线程优化主要减少CPU调度带来的波动，在线程优化的过程中，需要注意以下几点：控制线程数量-线程池、检查线程间的锁、防止依赖等等。

2. 系统调度优化

启动过程中，减少binder与AMS、WMS的通信；启动过程中，不要启动子进程，会使得SystemServer繁忙；除了Activity外的组件要谨慎处理，会占用Message通道；Application和Activifty的onCreate异步初始化代码。

3. GC优化

避免进行大量的字符串操作，特别是序列化和反序列化；频繁使用的对象考虑复用；转移到Native下实现。

4. IO优化

IO分网络IO和磁盘IO。启动过程中不建议网络IO；减少不必要的磁盘IO。

5. 主页面布局优化

减少冗余或者嵌套布局来降低结构；用ViewStub替代在启动过程中不需要显示的UI组件；使用自定义View替代复杂的View叠加。

6. 闲时调用

IdleHandler，当Handler空闲的时候才会被调用。可以将从服务器获取Token的任务放在IdleHandler中执行，或者把一些不重要的View加载放到IdleHandler中执行。

7. 应用瘦身

- Inspect Code
- 代码混淆
- 图片选择RGB565
- 接入资源混淆
- 减少DEX数量

### 10. Application初始化，如果有几百个SDK需要初始化，如果解决多线程的依赖和先后顺序？

### 11. 静态内部类A能调用父类中的变量吗

静态内部类不能使用外部类的非static成员变量和成员方法。

- 内部类

内部类的定义是A类中嵌套一个B类，这个B类称为内部类。有没有static关键词，可以分为静态内部类和非静态内部类。

- 静态内部类

使用static修饰，可以直接访问，而不需要实例化外部类就可以用。静态内部类访问外部类的成员变量，必须有外部类的实例；

- 非静态内部类

可以随意访问外部类的成员变量以及成员方法，即便使用private修饰。一般创建内部类的时候，一般需要创建该内部类的外部类的实例，再通过这个实例创建内部类。因此，非静态内部类是依附在外部类中。

### 12. 给出一个回文单链表，头节点head，如果判断该链表是不是回文链表。时间复杂度和空间复杂度分别是。