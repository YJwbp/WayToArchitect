# Android 架构、文件目录等基础

## Android架构

### 1、静态

<img src="http://gityuan.com/images/android-arch/android-stack.png" alt="android-stack" style="zoom: 33%;" />

上图所示，是官网提供的Android系统架构图，包含5大部分：

1. 应用程序

   > Android 随附一套用于电子邮件、短信、日历、互联网浏览和联系人等的核心应用。平台随附的应用与用户可以选择安装的应用一样，没有特殊状态。因此第三方应用可成为用户的默认网络浏览器、短信 Messenger 甚至默认键盘（有一些例外，例如系统的“设置”应用）。
   >
   > 系统应用可用作用户的应用，以及提供开发者可从其自己的应用访问的主要功能。例如，如果您的应用要发短信，您无需自己构建该功能，可以改为调用已安装的短信应用向您指定的接收者发送消息。

2. Java API 框架  

   > 您可通过以 Java 语言编写的 API 使用 Android OS 的整个功能集。这些 API 形成创建 Android 应用所需的构建块，它们可简化核心模块化系统组件和服务的重复使用，包括以下组件和服务：
   >
   > - 丰富、可扩展的视图系统，可用以构建应用的 UI，包括列表、网格、文本框、按钮甚至可嵌入的网络浏览器
   > - 资源管理器，用于访问非代码资源，例如本地化的字符串、图形和布局文件
   > - 通知管理器，可让所有应用在状态栏中显示自定义提醒
   > - Activity 管理器，用于管理应用的生命周期，提供常见的导航返回栈
   > - 内容提供程序，可让应用访问其他应用（例如“联系人”应用）中的数据或者共享其自己的数据
   >
   > 开发者可以完全访问 Android 系统应用使用的框架 API

3. 系统运行库 

   > 1. 原生 C/C++ 库 
   >
   >    许多核心 Android 系统组件和服务（例如 ART 和 HAL）构建自原生代码，需要以 C 和 C++ 编写的原生库。Android 平台提供 Java 框架 API 以向应用显示其中部分原生库的功能。例如，您可以通过 Android 框架的 Java OpenGL API 访问 OpenGL ES，以支持在应用中绘制和操作 2D 和 3D 图形。如果开发的是需要 C 或 C++ 代码的应用，可以使用 Android NDK 直接从原生代码访问某些原生平台库。
   >
   > 2. Android Runtime
   >
   >    1. 对于运行 Android 5.0（API 级别 21）或更高版本的设备，每个应用都在其自己的进程中运行，并且有其自己的 Android Runtime (ART) 实例。ART 编写为通过执行 DEX 文件在低内存设备上运行多个虚拟机，DEX 文件是一种专为 Android 设计的字节码格式，经过优化，使用的内存很少。编译工具链（例如 Jack）将 Java 源代码编译为 DEX 字节码，使其可在 Android 平台上运行。
   >
   >       ART 的部分主要功能包括：
   >
   >       * 预先 (AOT) 和即时 (JIT) 编译
   >       * 优化的垃圾回收 (GC)
   >       * 更好的调试支持，包括专用采样分析器、详细的诊断异常和崩溃报告，并且能够设置监视点以监控特定字段
   >
   >       在 Android 版本 5.0（API 级别 21）之前，Dalvik 是 Android Runtime。如果您的应用在 ART 上运行效果很好，那么它应该也可在 Dalvik 上运行，但反过来不一定。
   >
   >    2. Android 还包含一套核心运行时库，可提供 Java API 框架使用的 Java 编程语言大部分功能，包括一些 Java 8 语言功能。

4. 硬解抽象层

   > 硬件抽象层 (HAL) 提供标准界面，向更高级别的 Java API 框架显示设备硬件功能。HAL 包含多个库模块，其中每个模块都为特定类型的硬件组件实现一个界面，例如相机或蓝牙模块。当框架 API 要求访问设备硬件时，Android 系统将为该硬件组件加载库模块。

5. Linux内核

   > Android 平台的基础是 Linux 内核。例如，Android Runtime (ART) 依靠 Linux 内核来执行底层功能，例如线程和低层内存管理。使用 Linux 内核可让 Android 利用主要安全功能，并且允许设备制造商为著名的内核开发硬件驱动程序。



### 2、动态

来自（http://gityuan.com/android/）

![process_status](http://gityuan.com/images/android-arch/android-boot.jpg)



## 面试题

18、Android中App是如何沙箱化的,为何要这么做？

16、[说下安卓虚拟机和java虚拟机的原理和不同点](https://blog.csdn.net/jason0539/article/details/50440669)?（JVM、Davilk、ART三者的原理和区别） 

#### JVM 和Dalvik虚拟机的区别

JVM:.java -> javac -> .class -> jar -> .jar
    
架构: 堆和栈的架构.

DVM:.java -> javac -> .class -> dx.bat -> .dex

架构: 寄存器(cpu上的一块高速缓存)

#### Android2个虚拟机的区别（一个5.0之前，一个5.0之后）

什么是Dalvik：Dalvik是Google公司自己设计用于Android平台的Java虚拟机。Dalvik虚拟机是Google等厂商合作开发的Android移动设备平台的核心组成部分之一，它可以支持已转换为.dex(即Dalvik Executable)格式的Java应用程序的运行，.dex格式是专为Dalvik应用设计的一种压缩格式，适合内存和处理器速度有限的系统。Dalvik经过优化，允许在有限的内存中同时运行多个虚拟机的实例，并且每一个Dalvik应用作为独立的Linux进程执行。独立的进程可以防止在虚拟机崩溃的时候所有程序都被关闭。

什么是ART:Android操作系统已经成熟，Google的Android团队开始将注意力转向一些底层组件，其中之一是负责应用程序运行的Dalvik运行时。Google开发者已经花了两年时间开发更快执行效率更高更省电的替代ART运行时。ART代表Android Runtime,其处理应用程序执行的方式完全不同于Dalvik，Dalvik是依靠一个Just-In-Time(JIT)编译器去解释字节码。开发者编译后的应用代码需要通过一个解释器在用户的设备上运行，这一机制并不高效，但让应用能更容易在不同硬件和架构上运行。ART则完全改变了这套做法，在应用安装的时候就预编译字节码为机器语言，这一机制叫Ahead-Of-Time(AOT)编译。在移除解释代码这一过程后，应用程序执行将更有效率，启动更快。

ART优点：

- 系统性能的显著提升。
- 应用启动更快、运行更快、体验更流畅、触感反馈更及时。
- 更长的电池续航能力。
- 支持更低的硬件。

ART缺点：

- 更大的存储空间占用，可能会增加10%-20%。
- 更长的应用安装时间。

#### ART和Davlik中垃圾回收的区别？



#### apk打包流程

Android的包文件APK分为两个部分：代码和资源，所以打包方面也分为资源打包和代码打包两个方面，下面就来分析资源和代码的编译打包原理。

APK整体的的打包流程如下图所示：

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/native/vm/apk_package_flow.png)

具体说来：

- 通过AAPT工具进行资源文件（包括AndroidManifest.xml、布局文件、各种xml资源等）的打包，生成R.java文件。
- 通过AIDL工具处理AIDL文件，生成相应的Java文件。
- 通过Java Compiler编译R.java、Java接口文件、Java源文件，生成.class文件。
- 通过dex命令，将.class文件和第三方库中的.class文件处理生成classes.dex，该过程主要完成Java字节码转换成Dalvik字节码，压缩常量池以及清除冗余信息等工作。
- 通过ApkBuilder工具将资源文件、DEX文件打包生成APK文件。
- 通过Jarsigner工具，利用KeyStore对生成的APK文件进行签名。
- 如果是正式版的APK，还会利用ZipAlign工具进行对齐处理，对齐的过程就是将APK文件中所有的资源文件距离文件的起始距位置都偏移4字节的整数倍，这样通过内存映射访问APK文件的速度会更快，并且会减少其在设备上运行时的内存占用。

#### apk组成

- dex：最终生成的Dalvik字节码。
- res：存放资源文件的目录。
- asserts：额外建立的资源文件夹。
- lib：如果存在的话，存放的是ndk编出来的so库。   
- META-INF：存放签名信息

MANIFEST.MF（清单文件）：其中每一个资源文件都有一个SHA-256-Digest签名，MANIFEST.MF文件的SHA256（SHA1）并base64编码的结果即为CERT.SF中的SHA256-Digest-Manifest值。

CERT.SF（待签名文件）：除了开头处定义的SHA256（SHA1）-Digest-Manifest值，后面几项的值是对MANIFEST.MF文件中的每项再次SHA256并base64编码后的值。

CERT.RSA（签名结果文件）：其中包含了公钥、加密算法等信息。首先对前一步生成的MANIFEST.MF使用了SHA256（SHA1）-RSA算法，用开发者私钥签名，然后在安装时使用公钥解密。最后，将其与未加密的摘要信息（MANIFEST.MF文件）进行对比，如果相符，则表明内容没有被修改。

- androidManifest：程序的全局清单配置文件。
- resources.arsc：编译后的二进制资源文件。



#### 21、Asset目录与res目录的区别？

assets：不会在 R 文件中生成相应标记，存放到这里的资源在打包时会打包到程序安装包中。（通过 AssetManager 类访问这些文件）

res：会在 R 文件中生成 id 标记，资源在打包时如果使用到则打包到安装包中，未用到不会打入安装包中。

res/anim：存放动画资源。

res/raw：和 asset 下文件一样，打包时直接打入程序安装包中（会映射到 R 文件中）。

#### 13、Jar和Aar的区别

Jar包里面只有代码，aar里面不光有代码还包括资源文件，比如 drawable 文件，xml资源文件。对于一些不常变动的 Android Library，我们可以直接引用 aar，加快编译速度。





#### 62、AndroidManifest的作用与理解

AndroidManifest.xml文件，也叫清单文件，来获知应用中是否包含该组件，如果有会直接启动该组件。可以理解是一个应用的配置文件。

作用：

- 为应用的 Java 软件包命名。软件包名称充当应用的唯一标识符。
- 描述应用的各个组件，包括构成应用的 Activity、服务、广播接收器和内容提供程序。它还为实现每个组件的类命名并发布其功能，例如它们可以处理的 Intent - 消息。这些声明向 Android 系统告知有关组件以及可以启动这些组件的条件的信息。
- 确定托管应用组件的进程。
- 声明应用必须具备哪些权限才能访问 API 中受保护的部分并与其他应用交互。还声明其他应用与该应用组件交互所需具备的权限
- 列出 Instrumentation类，这些类可在应用运行时提供分析和其他信息。这些声明只会在应用处于开发阶段时出现在清单中，在应用发布之前将移除。
- 声明应用所需的最低 Android API 级别
- 列出应用必须链接到的库

## 匿名共享内存原理

https://www.jianshu.com/p/d9bc9c668ba6



# Handler机制

## 原理解析

### 1、结构图



![handler_java](http://gityuan.com/images/handler/handler_java.jpg)

**涉及角色：**

1. **Handler：**负责发送并处理消息
2. **Message：** 消息，Runnable也会被封装成Message
3. **MessageQueue：**消息队列，负责消息的存储与管理，链表实现
4. **Looper：**在Looper所在线程中，不断循环，取出消息，分发给Handler处理



**处理流程：**

1. Handler通过sendMessage()发送Message到MessageQueue队列；
2. Looper通过loop()，不断提取出达到触发条件的Message，并将Message交给target来处理；
3. 经过dispatchMessage()后，交回给Handler的handleMessage()来进行相应地处理。
4. 将Message加入MessageQueue时，处往管道写入字符，可以会唤醒loop线程；如果MessageQueue中没有Message，并处于Idle状态，则会执行IdelHandler接口中的方法，往往用于做一些清理性地工作。



### 2、源码解析

从使用的角度，一步步深入了解Handler相关源码



#### 2.1 创建Handler

要使用Handler，首先要创建个Handler对象。

但是，在非UI线程创建会崩溃！why？

```java
public Handler(Callback callback, boolean async) {
    mLooper = Looper.myLooper();
  	// 1、创建Handler之前需要保证Looper已经存在
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread " + Thread.currentThread()
                    + " that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}

	// Looper类
   public static @Nullable Looper myLooper() {
        return sThreadLocal.get();
    }
```

`解析：`首先要保证消息处理体系建立完成，然后才能通过Handler发送消息



好吧，那么我们就去prepare这个Looper



#### 2.2 prepare Looper

```java
// 1、通过prepare静态方法即可创建Looper
private static void prepare(boolean quitAllowed) {
  	// 2、每个线程只能有一个Looper，通过ThreadLocal来保证
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}

	// 3、Looper构造的时候，就创建好了相关联的MessageQueue；
	// 每个线程只能有一个Looper，同样也只能有一个MessageQueue
  private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
```

好了，prepare也调了，可以创建Handler了吧？

可以是可以，但是，现在只创建了对象，消息循环体系还没建立呢，没法处理消息



#### 2.3 开启消息循环

```java
public static void loop() {
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;

  	// 1、这里无限循环，取出消息
    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }

        try {
          	// 2、派发到这里进行处理，target即Handler
            msg.target.dispatchMessage(msg);
            dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
        } finally {
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }
     
      	// 3、消息回收，对应的我们也应该注意复用：Message.obtain()、handler.obtainMessage()
        msg.recycleUnchecked();
    }
}
```



好了，现在可以创建Handler，发送消息了



#### 2.4 发送消息



```java
// Handler 类
// 1、所有的发消息接口(post/sendMessage等)，最终都会到这里
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}
```



**Message怎么封装的呢？**

```java
// 1、 Runnable 封装到callback 
private static Message getPostMessage(Runnable r) {
    Message m = Message.obtain();
    m.callback = r;
    return m;
}

//2、延时：
public final boolean sendMessageDelayed(Message msg, long delayMillis){
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }

// 3、handler 封装到 target！！！
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
     msg.target = this;
     if (mAsynchronous) {
         msg.setAsynchronous(true);
     }
     return queue.enqueueMessage(msg, uptimeMillis);
 }
```



#### 2.5 处理消息

消息循环那里，不断取出消息，调用**dispatchMessage**处理消息

```java
// target 赋值来自于上节的 enqueueMessage
msg.target.dispatchMessage(msg);
```



那么，具体怎么处理消息的呢？

```java
// Handler 类
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}
```

**这里可以看到消息分发的优先级依次是：**

1. Message的回调方法：`message.callback.run()`，优先级最高；// 即Runnable
2. Handler的回调方法：`Handler.mCallback.handleMessage(msg)`，优先级仅次于1；// Handler创建时传入
3. Handler的默认方法：`Handler.handleMessage(msg)`，优先级最低。// 用户重写的方法





**消息缓存：**

为了提供效率，提供了一个大小为50的Message缓存队列，减少对象不断创建与销毁的过程。



## 高级部分

### IdleHandler

###同步屏障



## 面试题

####主线程的死循环一直运行是不是特别消耗CPU资源呢？

 其实不然，这里就涉及到Linux pipe/epoll机制，简单说就是在主线程的MessageQueue没有消息时，便阻塞在loop的queue.next()中的nativePollOnce()方法里，详情见[Android消息机制1-Handler(Java层)](https://link.zhihu.com/?target=http%3A//www.yuanhh.com/2015/12/26/handler-message-framework/%23next)，此时主线程会释放CPU资源进入休眠状态，直到下个消息到达或者有事务发生，通过往pipe管道写端写入数据来唤醒主线程工作。

这里采用的epoll机制，是一种IO多路复用机制，可以同时监控多个描述符，当某个描述符就绪(读或写就绪)，则立刻通知相应程序进行读或写操作，本质同步I/O，即读写是阻塞的。 所以说，主线程大多数时候都是处于休眠状态，并不会消耗大量CPU资源。





####非UI线程可以更新UI吗?

可以，当访问UI时，ViewRootImpl会调用checkThread方法去检查当前访问UI的线程是哪个，如果不是UI线程则会抛出异常。执行onCreate方法的那个时候ViewRootImpl还没创建，无法去检查当前线程.ViewRootImpl的创建在onResume方法回调之后。

    void checkThread() {
        if (mThread != Thread.currentThread()) {
            throw new CalledFromWrongThreadException(
                    "Only the original thread that created a view hierarchy can touch its views.");
        }
    }

非UI线程是可以刷新UI的，前提是它要拥有自己的ViewRoot,即更新UI的线程和创建ViewRoot的线程是同一个，或者在执行checkThread()前更新UI。



####Looper在主线程中死循环为什么没有导致界面的卡死？

1. 导致卡死的是在Ui线程中执行耗时操作导致界面出现掉帧，甚至`ANR`，`Looper.loop()`这个操作本身不会导致这个情况。
2. 有人可能会说，我在点击事件中设置死循环会导致界面卡死，同样都是死循环，不都一样的吗？Looper会在没有消息的时候阻塞当前线程，释放CPU资源，等到有消息到来的时候，再唤醒主线程。
3. App进程中是需要死循环的，如果循环结束的话，App进程就结束了。

*建议阅读：*

> [《Android中为什么主线程不会因为Looper.loop()里的死循环卡死？》](https://www.zhihu.com/question/34652589)

#### IdleHandler介绍？

介绍： IdleHandler是在Hanlder空闲时处理空闲任务的一种机制。

执行场景：

- `MessageQueue`没有消息，队列为空的时候。
- `MessageQueue`属于延迟消息，当前没有消息执行的时候。

会不会发生死循环： 答案是否定的，`MessageQueue`使用计数的方法保证一次调用`MessageQueue#next`方法只会使用一次的`IdleHandler`集合。



####postdelayed

如果在当前线程内使用Handler postdelayed 两个消息，一个延迟5s，一个延迟10s，然后使当前线程sleep 5秒，以上消息的执行时间会如何变化？

答：照常执行

扩展：sleep时间<=5 对两个消息无影响，5< sleep时间 <=10 对第一个消息有影响，第一个消息会延迟到sleep后执行，sleep时间>10 对两个时间都有影响，都会延迟到sleep后执行。



# 跨进程

27、怎么控制另外一个进程的View显示（RemoteView）？

#### 39、多进程场景遇见过么？

1、在新的进程中，启动前台Service，播放音乐。
2、一个成熟的应用一定是多模块化的。首先多进程开发能为应用解决了OOM问题，因为Android对内存的限制是针对于进程的，所以，当我们需要加载大图之类的操作，可以在新的进程中去执行，避免主进程OOM。而且假如图片浏览进程打开了一个过大的图片，java heap 申请内存失败，该进程崩溃并不影响我主进程的使用。



#### Android中进程和线程的关系？区别？

- 线程是CPU调度的最小单元，同时线程是一种有限的系统资源；而进程一般指一个执行单元，在PC和移动设备上指一个程序或者一个应用。
- 一般来说，一个App程序至少有一个进程，一个进程至少有一个线程（包含与被包含的关系），通俗来讲就是，在App这个工厂里面有一个进程，线程就是里面的生产线，但主线程（即主生产线）只有一条，而子线程（即副生产线）可以有多个。
- 进程有自己独立的地址空间，而进程中的线程共享此地址空间，都可以并发执行。




#### 如何开启多进程？应用是否可以开启N个进程？

在AndroidManifest中给四大组件指定属性android:process开启多进程模式，在内存允许的条件下可以开启N个进程。


#### 为何需要IPC？多进程通信可能会出现的问题？

所有运行在不同进程的四大组件（Activity、Service、Receiver、ContentProvider）共享数据都会失败，这是由于Android为每个应用分配了独立的虚拟机，不同的虚拟机在内存分配上有不同的地址空间，这会导致在不同的虚拟机中访问同一个类的对象会产生多份副本。比如常用例子（通过开启多进程获取更大内存空间、两个或者多个应用之间共享数据、微信全家桶）。

一般来说，使用多进程通信会造成如下几方面的问题:

- 静态成员和单例模式完全失效：独立的虚拟机造成。
- 线程同步机制完全失效：独立的虚拟机造成。
- SharedPreferences的可靠性下降：这是因为Sp不支持两个进程并发进行读写，有一定几率导致数据丢失。
- Application会多次创建：Android系统在创建新的进程时会分配独立的虚拟机，所以这个过程其实就是启动一个应用的过程，自然也会创建新的Application。


#### Android中IPC方式、各种方式优缺点？

![image](https://user-gold-cdn.xitu.io/2019/3/8/1695c1ab2aabf780?imageslim)


#### 讲讲AIDL？如何优化多模块都使用AIDL的情况？

AIDL(Android Interface Definition Language，Android接口定义语言)：如果在一个进程中要调用另一个进程中对象的方法，可使用AIDL生成可序列化的参数，AIDL会生成一个服务端对象的代理类，通过它客户端可以实现间接调用服务端对象的方法。

AIDL的本质是系统提供了一套可快速实现Binder的工具。关键类和方法：

- AIDL接口：继承IInterface。
- Stub类：Binder的实现类，服务端通过这个类来提供服务。
- Proxy类：服务端的本地代理，客户端通过这个类调用服务端的方法。
- asInterface()：客户端调用，将服务端返回的Binder对象，转换成客户端所需要的AIDL接口类型的对象。如果客户端和服务端位于同一进程，则直接返回Stub对象本身，否则返回系统封装后的Stub.proxy对象。
- asBinder()：根据当前调用情况返回代理Proxy的Binder对象。
- onTransact()：运行在服务端的Binder线程池中，当客户端发起跨进程请求时，远程请求会通过系统底层封装后交由此方法来处理。
- transact()：运行在客户端，当客户端发起远程请求的同时将当前线程挂起。之后调用服务端的onTransact()直到远程请求返回，当前线程才继续执行。

当有多个业务模块都需要AIDL来进行IPC，此时需要为每个模块创建特定的aidl文件，那么相应的Service就会很多。必然会出现系统资源耗费严重、应用过度重量级的问题。解决办法是建立Binder连接池，即将每个业务模块的Binder请求统一转发到一个远程Service中去执行，从而避免重复创建Service。

工作原理：每个业务模块创建自己的AIDL接口并实现此接口，然后向服务端提供自己的唯一标识和其对应的Binder对象。服务端只需要一个Service并提供一个queryBinder接口，它会根据业务模块的特征来返回相应的Binder对象，不同的业务模块拿到所需的Binder对象后就可以进行远程方法的调用了。


#### 为什么选择Binder？

为什么选用Binder，在讨论这个问题之前，我们知道Android也是基于Linux内核，Linux现有的进程通信手段有以下几种：

- 管道：在创建时分配一个page大小的内存，缓存区大小比较有限；
- 消息队列：信息复制两次，额外的CPU消耗；不合适频繁或信息量大的通信；
- 共享内存：无须复制，共享缓冲区直接附加到进程虚拟地址空间，速度快；但进程间的同步问题操作系统无法实现，必须各进程利用同步工具解决；
- 套接字：作为更通用的接口，传输效率低，主要用于不同机器或跨网络的通信；
- 信号量：常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。 不适用于信息交换，更适用于进程中断控制，比如非法内存访问，杀死某个进程等；

既然有现有的IPC方式，为什么重新设计一套Binder机制呢。主要是出于三个方面的考量：

- 1、效率：传输效率主要影响因素是内存拷贝的次数，拷贝次数越少，传输速率越高。从Android进程架构角度分析：对于消息队列、Socket和管道来说，数据先从发送方的缓存区拷贝到内核开辟的缓存区中，再从内核缓存区拷贝到接收方的缓存区，一共两次拷贝，如图：

![image](https://user-gold-cdn.xitu.io/2019/3/8/1695c427354bbec4?imageslim)

而对于Binder来说，数据从发送方的缓存区拷贝到内核的缓存区，而接收方的缓存区与内核的缓存区是映射到同一块物理地址的，节省了一次数据拷贝的过程，如图：

![image](https://user-gold-cdn.xitu.io/2019/3/8/1695c428e3c4a95a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

共享内存不需要拷贝，Binder的性能仅次于共享内存。

- 2、稳定性：上面说到共享内存的性能优于Binder，那为什么不采用共享内存呢，因为共享内存需要处理并发同步问题，容易出现死锁和资源竞争，稳定性较差。Socket虽然是基于C/S架构的，但是它主要是用于网络间的通信且传输效率较低。Binder基于C/S架构 ，Server端与Client端相对独立，稳定性较好。
- 3、安全性：传统Linux IPC的接收方无法获得对方进程可靠的UID/PID，从而无法鉴别对方身份；而Binder机制为每个进程分配了UID/PID，且在Binder通信时会根据UID/PID进行有效性检测。

#### Binder机制的作用和原理？

Linux系统将一个进程分为用户空间和内核空间。对于进程之间来说，用户空间的数据不可共享，内核空间的数据可共享，为了保证安全性和独立性，一个进程不能直接操作或者访问另一个进程，即Android的进程是相互独立、隔离的，这就需要跨进程之间的数据通信方式。普通的跨进程通信方式一般需要2次内存拷贝，如下图所示：

![image](https://user-gold-cdn.xitu.io/2019/3/8/1695c1ab41198d5c?imageslim)

一次完整的 Binder IPC 通信过程通常是这样：

- 首先 Binder 驱动在内核空间创建一个数据接收缓存区。
- 接着在内核空间开辟一块内核缓存区，建立内核缓存区和内核中数据接收缓存区之间的映射关系，以及内核中数据接收缓存区和接收进程用户空间地址的映射关系。
- 发送方进程通过系统调用 copyfromuser() 将数据 copy 到内核中的内核缓存区，由于内核缓存区和接收进程的用户空间存在内存映射，因此也就相当于把数据发送到了接收进程的用户空间，这样便完成了一次进程间的通信。

![image](https://user-gold-cdn.xitu.io/2019/3/8/1695c1ab2efe8dc5?imageslim)


#### Binder框架中ServiceManager的作用？

Binder框架 是基于 C/S 架构的。由一系列的组件组成，包括 Client、Server、ServiceManager、Binder驱动，其中 Client、Server、Service Manager 运行在用户空间，Binder 驱动运行在内核空间。如下图所示：

![image](https://user-gold-cdn.xitu.io/2019/3/8/1695c1ab50cf525f?imageslim)

- Server&Client：服务器&客户端。在Binder驱动和Service Manager提供的基础设施上，进行Client-Server之间的通信。
- ServiceManager（如同DNS域名服务器）服务的管理者，将Binder名字转换为Client中对该Binder的引用，使得Client可以通过Binder名字获得Server中Binder实体的引用。
- Binder驱动（如同路由器）：负责进程之间binder通信的建立，计数管理以及数据的传递交互等底层支持。

最后，结合[Android跨进程通信：图文详解 Binder机制 ](https://blog.csdn.net/carson_ho/article/details/73560642)的总结图来综合理解一下：

![image](https://user-gold-cdn.xitu.io/2019/3/8/1695c1ab5abdf775?imageslim)

#### Binder 的完整定义

- 从进程间通信的角度看，Binder 是一种进程间通信的机制；
- 从 Server 进程的角度看，Binder 指的是 Server 中的 Binder 实体对象；
- 从 Client 进程的角度看，Binder 指的是 Binder 代理对象，是 Binder 实体对象的一个远程代理;
- 从传输过程的角度看，Binder 是一个可以跨进程传输的对象；Binder 驱动会对这个跨越进程边界的对象对一点点特殊处理，自动完成代理对象和本地对象之间的转换。


#### 手写实现简化版AMS（AIDL实现）

与Binder相关的几个类的职责:

- IBinder：跨进程通信的Base接口，它声明了跨进程通信需要实现的一系列抽象方法，实现了这个接口就说明可以进行跨进程通信，Client和Server都要实现此接口。
- IInterface：这也是一个Base接口，用来表示Server提供了哪些能力，是Client和Server通信的协议。
- Binder：提供Binder服务的本地对象的基类，它实现了IBinder接口，所有本地对象都要继承这个类。
- BinderProxy：在Binder.java这个文件中还定义了一个BinderProxy类，这个类表示Binder代理对象它同样实现了IBinder接口，不过它的很多实现都交由native层处理。Client中拿到的实际上是这个代理对象。
- Stub：这个类在编译aidl文件后自动生成，它继承自Binder，表示它是一个Binder本地对象；它是一个抽象类，实现了IInterface接口，表明它的子类需要实现Server将要提供的具体能力（即aidl文件中声明的方法）。
- Proxy：它实现了IInterface接口，说明它是Binder通信过程的一部分；它实现了aidl中声明的方法，但最终还是交由其中的mRemote成员来处理，说明它是一个代理对象，mRemote成员实际上就是BinderProxy。

aidl文件只是用来定义C/S交互的接口，Android在编译时会自动生成相应的Java类，生成的类中包含了Stub和Proxy静态内部类，用来封装数据转换的过程，实际使用时只关心具体的Java接口类即可。为什么Stub和Proxy是静态内部类呢？这其实只是为了将三个类放在一个文件中，提高代码的聚合性。通过上面的分析，我们其实完全可以不通过aidl，手动编码来实现Binder的通信，下面我们通过编码来实现ActivityManagerService：

1、首先定义IActivityManager接口：


    public interface IActivityManager extends IInterface {
        //binder描述符
        String DESCRIPTOR = "android.app.IActivityManager";
        //方法编号
        int TRANSACTION_startActivity = IBinder.FIRST_CALL_TRANSACTION + 0;
        //声明一个启动activity的方法，为了简化，这里只传入intent参数
        int startActivity(Intent intent) throws RemoteException;
    }

 


2、然后，实现ActivityManagerService侧的本地Binder对象基类：


    // 名称随意，不一定叫Stub
    public abstract class ActivityManagerNative extends Binder implements IActivityManager {
    
        public static IActivityManager asInterface(IBinder obj) {
            if (obj == null) {
                return null;
            }
            IActivityManager in = (IActivityManager) obj.queryLocalInterface(IActivityManager.DESCRIPTOR);
            if (in != null) {
                return in;
            }
            //代理对象，见下面的代码
            return new ActivityManagerProxy(obj);
        }
    
        @Override
        public IBinder asBinder() {
            return this;
        }
    
        @Override
        protected boolean onTransact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
            switch (code) {
                // 获取binder描述符
                case INTERFACE_TRANSACTION:
                    reply.writeString(IActivityManager.DESCRIPTOR);
                    return true;
                // 启动activity，从data中反序列化出intent参数后，直接调用子类startActivity方法启动activity。
                case IActivityManager.TRANSACTION_startActivity:
                    data.enforceInterface(IActivityManager.DESCRIPTOR);
                    Intent intent = Intent.CREATOR.createFromParcel(data);
                    int result = this.startActivity(intent);
                    reply.writeNoException();
                    reply.writeInt(result);
                    return true;
            }
            return super.onTransact(code, data, reply, flags);
        }
    }


3、接着，实现Client侧的代理对象：


    public class ActivityManagerProxy implements IActivityManager {
        private IBinder mRemote;
    
        public ActivityManagerProxy(IBinder remote) {
            mRemote = remote;
        }
    
        @Override
        public IBinder asBinder() {
            return mRemote;
        }
    
        @Override
        public int startActivity(Intent intent) throws RemoteException {
            Parcel data = Parcel.obtain();
            Parcel reply = Parcel.obtain();
            int result;
            try {
                // 将intent参数序列化，写入data中
                intent.writeToParcel(data, 0);
                // 调用BinderProxy对象的transact方法，交由Binder驱动处理。
                mRemote.transact(IActivityManager.TRANSACTION_startActivity, data, reply, 0);
                reply.readException();
                // 等待server执行结束后，读取执行结果
                result = reply.readInt();
            } finally {
                data.recycle();
                reply.recycle();
            }
            return result;
        }
    }

 


4、最后，实现Binder本地对象（IActivityManager接口）：


    public class ActivityManagerService extends ActivityManagerNative {
        @Override
        public int startActivity(Intent intent) throws RemoteException {
            // 启动activity
            return 0;
        }
    }


简化版的ActivityManagerService到这里就已经实现了，剩下就是Client只需要获取到AMS的代理对象IActivityManager就可以通信了。

#### 简单讲讲 binder 驱动吧？

从 Java 层来看就像访问本地接口一样，客户端基于 BinderProxy 服务端基于 IBinder 对象，从 native 层来看来看客户端基于 BpBinder 到 ICPThreadState 到 binder 驱动，服务端由 binder 驱动唤醒 IPCThreadSate 到 BbBinder 。跨进程通信的原理最终是要基于内核的，所以最会会涉及到 binder_open 、binder_mmap 和 binder_ioctl这三种系统调用。


#### 跨进程传递大内存数据如何做？

binder 肯定是不行的，因为映射的最大内存只有 1M-8K，可以采用 binder + 匿名共享内存的形式，像跨进程传递大的 bitmap 需要打开系统底层的 ashmem 机制。


请按顺序仔细阅读下列文章提升对Binder机制的理解程度：

[写给 Android 应用工程师的 Binder 原理剖析](https://juejin.im/post/5acccf845188255c3201100f)

[Binder学习指南](http://weishu.me/2016/01/12/binder-index-for-newer/)

[Binder设计与实现](https://blog.csdn.net/universus/article/details/6211589)

[老罗Binder机制分析系列或Android系统源代码情景分析Binder章节](



# Binder机制

具体参考：

[https://github.com/YJwbp/WayToArchitect/blob/master/framework%E5%8E%9F%E7%90%86/Binder%E6%9C%BA%E5%88%B6.md](https://github.com/YJwbp/WayToArchitect/blob/master/framework原理/Binder机制.md)



# 启动相关

[看这里]([https://github.com/YJwbp/WayToArchitect/blob/master/framework%E5%8E%9F%E7%90%86/%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B.md](https://github.com/YJwbp/WayToArchitect/blob/master/framework原理/启动过程.md))

#### # Apk的安装过程？

*建议阅读：*

> [《Android Apk安装过程分析》](https://www.jianshu.com/p/953475cea991)

#### # Activity启动过程跟Window的关系？

*建议阅读：*

> [《简析Window、Activity、DecorView以及ViewRoot之间的错综关系》](https://www.jianshu.com/p/8766babc40e0)

#### #  Activity、Window、ViewRoot和DecorView之间的关系？

*建议阅读：*

> [《总结UI原理和高级的UI优化方式》](https://juejin.im/post/5dac6aa2518825630e5d17da)

### 

### 14、大体说清一个应用程序安装到手机上时发生了什么？

APK的安装流程如下所示：

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/app/package/apk_install_structure.png)

复制APK到/data/app目录下，解压并扫描安装包。

资源管理器解析APK里的资源文件。

解析AndroidManifest文件，并在/data/data/目录下创建对应的应用数据目录。

然后对dex文件进行优化，并保存在dalvik-cache目录下。

将AndroidManifest文件解析出的四大组件信息注册到PackageManagerService中。

安装完成后，发送广播。



### 10、说下四大组件的启动过程，四大组件的启动与销毁的方式。


#### 广播发送和接收的原理了解吗？

- 继承BroadcastReceiver，重写onReceive()方法。
- 通过Binder机制向ActivityManagerService注册广播。
- 通过Binder机制向ActivityMangerService发送广播。
- ActivityManagerService查找符合相应条件的广播（IntentFilter/Permission）的BroadcastReceiver，将广播发送到BroadcastReceiver所在的消息队列中。
- BroadcastReceiver所在消息队列拿到此广播后，回调它的onReceive()方法。



### 8、App启动流程（Activity的冷启动流程）。

点击应用图标后会去启动应用的Launcher Activity，如果Launcer Activity所在的进程没有创建，还会创建新进程，整体的流程就是一个Activity的启动流程。

Activity的启动流程图（放大可查看）如下所示：

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/app/component/activity_start_flow.png)

整个流程涉及的主要角色有：

- Instrumentation: 监控应用与系统相关的交互行为。
- AMS：组件管理调度中心，什么都不干，但是什么都管。
- ActivityStarter：Activity启动的控制器，处理Intent与Flag对Activity启动的影响，具体说来有：1 寻找符合启动条件的Activity，如果有多个，让用户选择；2 校验启动参数的合法性；3 返回int参数，代表Activity是否启动成功。
- ActivityStackSupervisior：这个类的作用你从它的名字就可以看出来，它用来管理任务栈。
- ActivityStack：用来管理任务栈里的Activity。
- ActivityThread：最终干活的人，Activity、Service、BroadcastReceiver的启动、切换、调度等各种操作都在这个类里完成。

注：这里单独提一下ActivityStackSupervisior，这是高版本才有的类，它用来管理多个ActivityStack，早期的版本只有一个ActivityStack对应着手机屏幕，后来高版本支持多屏以后，就有了多个ActivityStack，于是就引入了ActivityStackSupervisior用来管理多个ActivityStack。

整个流程主要涉及四个进程：

- 调用者进程，如果是在桌面启动应用就是Launcher应用进程。
- ActivityManagerService等待所在的System Server进程，该进程主要运行着系统服务组件。
- Zygote进程，该进程主要用来fork新进程。
- 新启动的应用进程，该进程就是用来承载应用运行的进程了，它也是应用的主线程（新创建的进程就是主线程），处理组件生命周期、界面绘制等相关事情。

有了以上的理解，整个流程可以概括如下：

- 1、点击桌面应用图标，Launcher进程将启动Activity（MainActivity）的请求以Binder的方式发送给了AMS。
- 2、AMS接收到启动请求后，交付ActivityStarter处理Intent和Flag等信息，然后再交给ActivityStackSupervisior/ActivityStack 处理Activity进栈相关流程。同时以Socket方式请求Zygote进程fork新进程。
- 3、Zygote接收到新进程创建请求后fork出新进程。
- 4、在新进程里创建ActivityThread对象，新创建的进程就是应用的主线程，在主线程里开启Looper消息循环，开始处理创建Activity。
- 5、ActivityThread利用ClassLoader去加载Activity、创建Activity实例，并回调Activity的onCreate()方法，这样便完成了Activity的启动。

最后，再看看另一幅启动流程图来加深理解：

![image](http://img.mp.itc.cn/upload/20170329/ca9567ce3bf04c4abdb4d124cebfee76_th.jpeg)

### 5、Android系统启动流程是什么？（提示：init进程 -> Zygote进程 –> SystemServer进程 –> 各种系统服务 –> 应用进程）

Android系统启动的核心流程如下：

- 1、**启动电源以及系统启动**：当电源按下时引导芯片从预定义的地方（固化在ROM）开始执行，加载引导程序BootLoader到RAM，然后执行。
- 2、**引导程序BootLoader**：BootLoader是在Android系统开始运行前的一个小程序，主要用于把系统OS拉起来并运行。
- 3、**Linux内核启动**：当内核启动时，设置缓存、被保护存储器、计划列表、加载驱动。当其完成系统设置时，会先在系统文件中寻找init.rc文件，并启动init进程。
- 4、**init进程启动**：初始化和启动属性服务，并且启动Zygote进程。
- 5、**Zygote进程启动**：创建JVM并为其注册JNI方法，创建服务器端Socket，启动SystemServer进程。
- 6、**SystemServer进程启动**：启动Binder线程池和SystemServiceManager，并且启动各种系统服务。
- 7、**Launcher启动**：被SystemServer进程启动的AMS会启动Launcher，Launcher启动后会将已安装应用的快捷图标显示到系统桌面上。

#### 需要更详细的分析请查看以下系列文章：

[Android系统启动流程之init进程启动](https://jsonchao.github.io/2019/02/18/Android%E7%B3%BB%E7%BB%9F%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B%E4%B9%8Binit%E8%BF%9B%E7%A8%8B%E5%90%AF%E5%8A%A8/)

[Android系统启动流程之Zygote进程启动](https://jsonchao.github.io/2019/02/24/Android%E7%B3%BB%E7%BB%9F%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B%E4%B9%8BZygote%E8%BF%9B%E7%A8%8B%E5%90%AF%E5%8A%A8/)

[Android系统启动流程之SystemServer进程启动](https://jsonchao.github.io/2019/03/03/Android%E7%B3%BB%E7%BB%9F%E5%90%AF%E5%8A%A8%E6%B5%81%E4%B9%8BSystemServer%E8%BF%9B%E7%A8%8B%E5%90%AF%E5%8A%A8/)

[Android系统启动流程之Launcher进程启动](https://jsonchao.github.io/2019/03/09/Android%E7%B3%BB%E7%BB%9F%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B%E4%B9%8BLauncher%E8%BF%9B%E7%A8%8B%E5%90%AF%E5%8A%A8/)

#### 系统是怎么帮我们启动找到桌面应用的？

通过意图，PMS 会解析所有 apk 的 AndroidManifest.xml ，如果解析过会存到 package.xml 中不会反复解析，PMS 有了它就能找到了。


### 6、启动一个程序，可以主界面点击图标进入，也可以从一个程序中跳转过去，二者有什么区别？

是因为启动程序（主界面也是一个app），发现了在这个程序中存在一个设置为<category android:name="android.intent.category.LAUNCHER" />的activity,
所以这个launcher会把icon提出来，放在主界面上。当用户点击icon的时候，发出一个Intent：
    

    Intent intent = mActivity.getPackageManager().getLaunchIntentForPackage(packageName);
    mActivity.startActivity(intent);   

跳过去可以跳到任意允许的页面，如一个程序可以下载，那么真正下载的页面可能不是首页（也有可能是首页），这时还是构造一个Intent，startActivity。这个intent中的action可能有多种view，download都有可能。系统会根据第三方程序向系统注册的功能，为你的Intent选择可以打开的程序或者页面。所以唯一的一点
不同的是从icon的点击启动的intent的action是相对单一的，从程序中跳转或者启动可能样式更多一些。本质是相同的。



# 屏幕刷新机制

总体流程、

源码分析

https://www.jianshu.com/p/10db590ed9a6 这个有些总结不错



### 26、TextView调用setText方法的内部执行流程。

Canvas的底层机制，绘制框架，硬件加速是什么原理，canvas lock的缓冲区是怎么回事？



# AmS、PMS、WmS等

各种交互过程中涉及到的类！！

deeplink 原理：https://www.jianshu.com/p/eb20492acf25

十分钟了解Android触摸事件原理（InputManagerService）：https://www.jianshu.com/p/f05d6b05ba17



## 类介绍

###AmS

AMS（ActivityManagerService）
ActivityManager是客户端用来管理系统中正在运行的所有Activity包括Task、Memory、Service等信息的工具。但是这些这些信息的维护工作却不是又ActivityManager负责的。在ActivityManager中有大量的get()方法，那么也就说明了他只是提供信息给AMS，由AMS去完成交互和调度工作。

**作用：**
统一调度所有应用程序的Activity的生命周期
启动或杀死应用程序的进程
启动并调度Service的生命周期
注册BroadcastReceiver，并接收和分发Broadcast
启动并发布ContentProvider
调度task
处理应用程序的Crash
查询系统当前运行状态




### 9、ActivityThread工作原理。

### 11、AMS是如何管理Activity的？



**ActivityThread**：是Android应用的主线程（UI线程）

### 7、AMS家族重要术语解释。

1.ActivityManagerServices，简称AMS，服务端对象，负责系统中所有Activity的生命周期。

2.ActivityThread，App的真正入口。当开启App之后，调用main()开始运行，开启消息循环队列，这就是传说的UI线程或者叫主线程。与ActivityManagerService一起完成Activity的管理工作。

3.ApplicationThread，用来实现ActivityManagerServie与ActivityThread之间的交互。在ActivityManagerSevice需要管理相关Application中的Activity的生命周期时，通过ApplicationThread的代理对象与ActivityThread通信。

4.ApplicationThreadProxy，是ApplicationThread在服务器端的代理，负责和客户端的ApplicationThread通信。AMS就是通过该代理与ActivityThread进行通信的。

5.Instrumentation，每一个应用程序只有一个Instrumetation对象，每个Activity内都有一个对该对象的引用，Instrumentation可以理解为应用进程的管家，ActivityThread要创建或暂停某个Activity时，都需要通过Instrumentation来进行具体的操作。

6.ActivityStack，Activity在AMS的栈管理，用来记录经启动的Activity的先后关系，状态信息等。通过ActivtyStack决定是否需要启动新的进程。

7.ActivityRecord，ActivityStack的管理对象，每个Acivity在AMS对应一个ActivityRecord，来记录Activity状态以及其他的管理信息。其实就是服务器端的Activit对象的映像。

8.TaskRecord，AMS抽象出来的一个“任务”的概念，是记录ActivityRecord的栈，一个“Task”包含若干个ActivityRecord。AMS用TaskRecord确保Activity启动和退出的顺序。如果你清楚Activity的4种launchMode，那么对这概念应该不陌生。



### WmS

WMS(WindowManagerService)：管理的整个系统所有窗口的UI（显示、隐藏、位置等）
作用:

* 为所有窗口分配Surface：客户端向WMS添加一个窗口的过程，其实就是WMS为其分配一块Surface的过程，一块块Surface在WMS的管理下有序的排布在屏幕上。Window的本质就是Surface。（简单的说Surface对应了一块屏幕缓冲区）

* 管理Surface的显示顺序、尺寸、位置

* 管理窗口动画

* 输入系统相关：派发系统按键和触摸消息

  > 当接收到一个触摸事件，它需要寻找一个最合适的窗口来处理消息，而WMS是窗口的管理者，系统中所有的窗口状态和信息都在其掌握之中，完成这一工作不在话下。



Window添加流程：https://www.jianshu.com/p/40776c123adb



####面试题




### 12、理解Window和WindowManager。

1.Window用于显示View和接收各种事件，Window有三种型：应用Window(每个Activity对应一个Window)、子Widow(不能单独存在，附属于特定Window)、系统window(toast和状态栏)

2.Window分层级，应用Window在1-99、子Window在1000-1999、系统Window在2000-2999.WindowManager提供了增改View的三个功能。

3.Window是个抽象概念：每一个Window对应着一个ViewRootImpl，Window通过ViewRootImpl来和View建立联系，View是Window存在的实体，只能通过WindowManager来访问Window。

4.WindowManager的实现是WindowManagerImpl，其再委托WindowManagerGlobal来对Window进行操作，其中有四种List分别储存对应的View、ViewRootImpl、WindowManger.LayoutParams和正在被删除的View。

5.Window的实体是存在于远端的WindowMangerService，所以增删改Window在本端是修改上面的几个List然后通过ViewRootImpl重绘View，通过WindowSession(每Window个对应一个)在远端修改Window。

6.Activity创建Window：Activity会在attach()中创建Window并设置其回调(onAttachedToWindow()、dispatchTouchEvent())，Activity的Window是由Policy类创建PhoneWindow实现的。然后通过Activity#setContentView()调用PhoneWindow的setContentView。


### 13、WMS是如何管理Window的？

#####非UI线程可以更新UI吗?

可以，当访问UI时，ViewRootImpl会调用checkThread方法去检查当前访问UI的线程是哪个，如果不是UI线程则会抛出异常。执行onCreate方法的那个时候ViewRootImpl还没创建，无法去检查当前线程.ViewRootImpl的创建在onResume方法回调之后。

    void checkThread() {
        if (mThread != Thread.currentThread()) {
            throw new CalledFromWrongThreadException(
                    "Only the original thread that created a view hierarchy can touch its views.");
        }
    }

非UI线程是可以刷新UI的，前提是它要拥有自己的ViewRoot,即更新UI的线程和创建ViewRoot的线程是同一个，或者在执行checkThread()前更新UI。

#####DecorView被加载到Window的过程

- 从Activity的startActivity开始，最终调用到ActivityThread的handleLaunchActivity方法来创建Activity，首先，会调用performLaunchActivity方法，内部会执行Activity的onCreate方法，从而完成DecorView和Activity的创建。然后，会调用handleResumeActivity，里面首先会调用performResumeActivity去执行Activity的onResume()方法，执行完后会得到一个ActivityClientRecord对象，然后通过r.window.getDecorView()的方式得到DecorView，然后会通过a.getWindowManager()得到WindowManager，最终调用其addView()方法将DecorView加进去。
- WindowManager的实现类是WindowManagerImpl，它内部会将addView的逻辑委托给WindowManagerGlobal，可见这里使用了接口隔离和委托模式将实现和抽象充分解耦。在WindowManagerGlobal的addView()方法中不仅会将DecorView添加到Window中，同时会创建ViewRootImpl对象，并将ViewRootImpl对象和DecorView通过root.setView()把DecorView加载到Window中。这里的ViewRootImpl是ViewRoot的实现类，是连接WindowManager和DecorView的纽带。View的三大流程均是通过ViewRoot来完成的。



19、[一个图片在app中调用R.id后是如何找到的](https://my.oschina.net/u/255456/blog/608229)？

