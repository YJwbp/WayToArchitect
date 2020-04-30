##是什么？

ANR(Application Not responding)，是指应用程序未响应，Android系统对于一些事件需要在一定的时间范围内完成，如果超过预定时间能未能得到有效响应或者响应时间过长，都会造成ANR。

一般地，这时往往会弹出一个提示框，告知用户当前xxx未响应，用户可选择继续等待或者强制关闭。



##为什么？

绝大多数人对ANR的了解仅停留在主线程耗时或CPU繁忙会导致ANR。面试过无数的候选人，几乎没有人能真正从系统级去梳理清晰ANR的来龙去脉，比如有哪些路径会引发ANR? 有没有可能主线程不耗时也出现ANR？如何更好的调试ANR?



那么哪些场景会造成ANR呢？

- Service Timeout：服务在规定时间内未启动完成；
- BroadcastQueue Timeout：广播在规定时间内未执行完成
- ContentProvider Timeout：provider首次启动超时
- InputDispatching Timeout：输入事件分发超时5s，包括按键和触摸事件。



### 场景

* 主线程被其他线程lock，导致死锁
* 主线程做耗时的操作：比如数据库读写。
* 本进程/其他进程的CPU占用率高
* binder数据量过大、binder 通信失败、主线程等待进程通信响应
* 



##怎么预防？

避开上述ANR的这几种场景即可。

作为应用开发者

* 应让主线程尽量只做UI相关的操作，避免耗时操作，比如过度复杂的UI绘制，网络操作，文件IO操作；
* 避免主线程跟工作线程发生锁的竞争，
* 减少系统耗时binder的调用，
* 谨慎使用sharePreference，
* 注意主线程执行provider query操作。

简而言之，尽可能减少主线程的负载，让其空闲待命，以期可随时响应用户的操作。



##怎么定位？

首先参照**ANR爆炸现场**一节，知道发生ANR时，系统的操作。



以下定位过程参考：https://juejin.im/post/5be698d4e51d452acb74ea4c

###一、 查看events_log



####1、`am_anr` 定位时间点、进程、ANR类型等

查看mobilelog文件夹下的events_log,从日志中搜索关键字：`am_anr`，找到出现ANR的时间点、进程PID、ANR类型。

如日志：

```
07-20 15:36:36.472  1000  1520  1597 I am_anr  : [0,1480,com.xxxx.moblie,952680005,Input dispatching timed out (AppWindowToken{da8f666 token=Token{5501f51 ActivityRecord{15c5c78 u0 com.xxxx.moblie/.ui.MainActivity t3862}}}, Waiting because no window has focus but there is a focused application that may eventually add a window when it finishes starting up.)]
复制代码
```

从上面的log我们可以看出： 应用`com.xxxx.moblie` 在`07-20 15:36:36.472`时间，发生了一次`KeyDispatchTimeout`类型的ANR，它的进程号是`1480`. 把关键的信息整理一下：
 **ANR时间**：07-20 15:36:36.472
 **进程pid**：1480
 **进程名**：com.xxxx.moblie
 **ANR类型**：KeyDispatchTimeout

#### 2、逆推时间

我们已经知道了发生`KeyDispatchTimeout`的ANR是因为 `input事件在5秒内没有处理完成`。那么在这个时间`07-20 15:36:36.472` 的前5秒，也就是（`15:36:30 ~15:36:31`）时间段左右程序到底做了什么事情？这个简单，因为我们已经知道pid了，再搜索一下`pid = 1480`的日志.这些日志表示该进程所运行的轨迹，关键的日志如下：

```
07-20 15:36:29.749 10102  1480  1737 D moblie-Application: [Thread:17329] receive an intent from server, action=com.ttt.push.RECEIVE_MESSAGE
07-20 15:36:30.136 10102  1480  1737 D moblie-Application: receiving an empty message, drop
07-20 15:36:35.791 10102  1480  1766 I Adreno  : QUALCOMM build                   : 9c9b012, I92eb381bc9
07-20 15:36:35.791 10102  1480  1766 I Adreno  : Build Date                       : 12/31/17
07-20 15:36:35.791 10102  1480  1766 I Adreno  : OpenGL ES Shader Compiler Version: EV031.22.00.01
07-20 15:36:35.791 10102  1480  1766 I Adreno  : Local Branch                     : 
07-20 15:36:35.791 10102  1480  1766 I Adreno  : Remote Branch                    : refs/tags/AU_LINUX_ANDROID_LA.UM.6.4.R1.08.00.00.309.049
07-20 15:36:35.791 10102  1480  1766 I Adreno  : Remote Branch                    : NONE
07-20 15:36:35.791 10102  1480  1766 I Adreno  : Reconstruct Branch               : NOTHING
07-20 15:36:35.826 10102  1480  1766 I vndksupport: sphal namespace is not configured for this process. Loading /vendor/lib64/hw/gralloc.msm8998.so from the current namespace instead.
07-20 15:36:36.682 10102  1480  1480 W ViewRootImpl[MainActivity]: Cancelling event due to no window focus: KeyEvent { action=ACTION_UP, keyCode=KEYCODE_PERIOD, scanCode=0, metaState=0, flags=0x28, repeatCount=0, eventTime=16099429, downTime=16099429, deviceId=-1, source=0x101 }
复制代码
```

从上面我们可以知道，在时间 07-20 15:36:29.749 程序收到了一个action消息。

```
07-20 15:36:29.749 10102  1480  1737 D moblie-Application: [Thread:17329] receive an intent from server, action=com.ttt.push.RECEIVE_MESSAGE。
复制代码
```

原来是应用`com.xxxx.moblie` 收到了一个推送消息（`com.ttt.push.RECEIVE_MESSAGE`）导致了阻塞，我们再串联一下目前所获取到的信息：在时间`07-20 15:36:29.749` 应用`com.xxxx.moblie` 收到了一下推送信息`action=com.ttt.push.RECEIVE_MESSAGE`发生阻塞，5秒后发生了`KeyDispatchTimeout的ANR`。



####3、CPU资源

虽然知道了是怎么开始的，但是具体原因还没有找到，是不是当时CPU很紧张、各路APP再抢占资源？ 我们再看看CPU的信息,。**搜索关键字关键字： `ANR IN`**

```
07-20 15:36:58.711  1000  1520  1597 E ActivityManager: ANR in com.xxxx.moblie (com.xxxx.moblie/.ui.MainActivity) (进程名)
07-20 15:36:58.711  1000  1520  1597 E ActivityManager: PID: 1480 (进程pid)
07-20 15:36:58.711  1000  1520  1597 E ActivityManager: Reason: Input dispatching timed out (AppWindowToken{da8f666 token=Token{5501f51 ActivityRecord{15c5c78 u0 com.xxxx.moblie/.ui.MainActivity t3862}}}, Waiting because no window has focus but there is a focused application that may eventually add a window when it finishes starting up.)
07-20 15:36:58.711  1000  1520  1597 E ActivityManager: Load: 0.0 / 0.0 / 0.0 (Load表明是1分钟,5分钟,15分钟CPU的负载)
07-20 15:36:58.711  1000  1520  1597 E ActivityManager: CPU usage from 20ms to 20286ms later (2018-07-20 15:36:36.170 to 2018-07-20 15:36:56.436):
07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   42% 6774/pressure: 41% user + 1.4% kernel / faults: 168 minor
07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   34% 142/kswapd0: 0% user + 34% kernel
07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   31% 1520/system_server: 13% user + 18% kernel / faults: 58724 minor 1585 major
07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   13% 29901/com.ss.android.article.news: 7.7% user + 6% kernel / faults: 56007 minor 2446 major
07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   13% 32638/com.android.quicksearchbox: 9.4% user + 3.8% kernel / faults: 48999 minor 1540 major
07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   11% (CPU的使用率)1480/com.xxxx.moblie: 5.2%(用户态的使用率) user + (内核态的使用率) 6.3% kernel / faults: 76401 minor 2422 major
07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   8.2% 21000/kworker/u16:12: 0% user + 8.2% kernel
07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   0.8% 724/mtd: 0% user + 0.8% kernel / faults: 1561 minor 9 major
07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   8% 29704/kworker/u16:8: 0% user + 8% kernel
07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   7.9% 24391/kworker/u16:18: 0% user + 7.9% kernel
07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   7.1% 30656/kworker/u16:14: 0% user + 7.1% kernel
07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   7.1% 9998/kworker/u16:4: 0% user + 7.1% kernel
复制代码
```

我已经在log 中标志了相关的含义。`com.xxxx.moblie` 占用了11%的CPU，其实这并不算多。现在的手机基本都是多核CPU。假如你的CPU是4核，那么上限是400%，以此类推。

既然不是CPU负载的原因，那么到底是什么原因呢？ 这时就要看我们的终极大杀器——`traces.txt`。



###二、 traces.txt 日志分析

####1、抓取traces文件

当APP不响应、响应慢了、或者WatchDog的监视没有得到回应时，系统就会dump出一个`traces.txt`文件，存放在文件目录:`/data/anr/traces.txt`，通过traces文件,我们可以拿到线程名、堆栈信息、线程当前状态、binder call等信息。
 通过adb命令拿到该文件：`adb pull /data/anr/traces.txt`
 trace: Cmd line:com.xxxx.moblie

```
"main" prio=5 tid=1 Runnable
  | group="main" sCount=0 dsCount=0 obj=0x73bcc7d0 self=0x7f20814c00
  | sysTid=20176 nice=-10 cgrp=default sched=0/0 handle=0x7f251349b0
  | state=R schedstat=( 0 0 0 ) utm=12 stm=3 core=5 HZ=100
  | stack=0x7fdb75e000-0x7fdb760000 stackSize=8MB
  | held mutexes= "mutator lock"(shared held)
  // java 堆栈调用信息,可以查看调用的关系，定位到具体位置
  at ttt.push.InterceptorProxy.addMiuiApplication(InterceptorProxy.java:77)
  at ttt.push.InterceptorProxy.create(InterceptorProxy.java:59)
  at android.app.Activity.onCreate(Activity.java:1041)
  at miui.app.Activity.onCreate(SourceFile:47)
  at com.xxxx.moblie.ui.b.onCreate(SourceFile:172)
  at com.xxxx.moblie.ui.MainActivity.onCreate(SourceFile:68)
  at android.app.Activity.performCreate(Activity.java:7050)
  at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1214)
  at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2807)
  at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2929)
  at android.app.ActivityThread.-wrap11(ActivityThread.java:-1)
  at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1618)
  at android.os.Handler.dispatchMessage(Handler.java:105)
  at android.os.Looper.loop(Looper.java:171)
  at android.app.ActivityThread.main(ActivityThread.java:6699)
  at java.lang.reflect.Method.invoke(Native method)
  at com.android.internal.os.Zygote$MethodAndArgsCaller.run(Zygote.java:246)
  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:783)
复制代码
```

我详细解析一下`traces.txt`里面的一些字段，看看它到底能给我们提供什么信息.
 **main**：main标识是主线程，如果是线程，那么命名成“Thread-X”的格式,x表示线程id,逐步递增。
 **prio**：线程优先级,默认是5
 **tid**：tid不是线程的id，是线程唯一标识ID
 **group**：是线程组名称
 **sCount**：该线程被挂起的次数
 **dsCount**：是线程被调试器挂起的次数
 **obj**：对象地址
 **self**：该线程Native的地址
 **sysTid**：是线程号(主线程的线程号和进程号相同)
 **nice**：是线程的调度优先级
 **sched**：分别标志了线程的调度策略和优先级
 **cgrp**：调度归属组
 **handle**：线程处理函数的地址。
 **state**：是调度状态
 **schedstat**：从 `/proc/[pid]/task/[tid]/schedstat`读出，三个值分别表示线程在cpu上执行的时间、线程的等待时间和线程执行的时间片长度，不支持这项信息的三个值都是0；
 **utm**：是线程用户态下使用的时间值(单位是jiffies）
 **stm**：是内核态下的调度时间值
 **core**：是最后执行这个线程的cpu核的序号。

Java的堆栈信息是我们最关心的，它能够定位到具体位置。从上面的traces,我们可以判断

`ttt.push.InterceptorProxy.addMiuiApplicationInterceptorProxy.java:77` 导致了`com.xxxx.moblie`发生了ANR。这时候可以对着源码查看，找到出问题，并且解决它。



总结一下这分析流程：首先我们搜索`am_anr`，找到出现ANR的时间点、进程PID、ANR类型、然后再找搜索`PID`，找前5秒左右的日志。过滤ANR IN 查看CPU信息，接着查看`traces.txt`，找到java的堆栈信息定位代码位置，最后查看源码，分析与解决问题。这个过程基本能找到发生ANR的来龙去脉。

#### 2、了解线程状态

![image-20200430172123592](https://tva1.sinaimg.cn/large/007S8ZIlly1gebx17xr6tj311i0kyaqz.jpg)



#### 3、dropbox日志

dropbox文件目录路径一般是`/data/system/dropbox/`

正式版的机器我们是没有读取这个目录的权限的，即使你拥有系统权限也不行。

DropBoxManager 是 Android 在 Froyo(API level 8) 引入的用来持续化存储系统数据的机制, 主要用于记录 Android 运行过程中, 内核, 系统进程, 用户进程等出现严重问题时的 log, 可以认为这是一个可持续存储的系统级别的 logcat.

可以记录如下类型的log：

* crash/anr/strict_mode/lowmem/BATTERY_DISCHARGE_INFO/开机/系统/Native 进程的崩溃等待



在dropbox在日志存储时会发送系统广播：**DropBoxManager.ACTION_DROPBOX_ENTRY_ADDED**

app可以接收监听改广播获取指定的数据文件内容，内容发送到指定的服务器或邮箱完成错误自动上报

**利用前提：app要具有系统权限** **因为：**DropBoxManager.ACTION_DROPBOX_ENTRY_ADDED是保护性广播，且读取系统日志也需要android.Manifest.permission.READ_LOGS权限



```java
DropboxManager dropboxManager = (DropboxManager ) mContext.getSystemService(Context.DROPBOX_SERVICE)
```



## 底层原理？

1. 彻底理解安卓应用无响应机制：http://gityuan.com/2019/04/06/android-anr/
2. Android ANR的触发源码分析：http://gityuan.com/2016/07/02/android-anr



### ANR触发机制

#### service超时机制

下面来看看埋炸弹与拆炸弹在整个服务启动(startService)过程所处的环节。

![service_anr](http://gityuan.com/images/android-anr/service_anr.jpg)

图解1：

1. 客户端(App进程)向中控系统(system_server进程)发起启动服务的请求
2. 中控系统派出一名空闲的通信员(binder_1线程)接收该请求，紧接着向组件管家(ActivityManager线程)发送消息，埋下定时炸弹
3. 通讯员1号(binder_1)通知工地(service所在进程)的通信员准备开始干活
4. 通讯员3号(binder_3)收到任务后转交给包工头(main主线程)，加入包工头的任务队列(MessageQueue)
5. 包工头经过一番努力干完活(完成service启动的生命周期)，然后等待SharedPreferences(简称SP)的持久化；
6. 包工头在SP执行完成后，立刻向中控系统汇报工作已完成
7. 中控系统的通讯员2号(binder_2)收到包工头的完工汇报后，立刻拆除炸弹。如果在炸弹倒计时结束前拆除炸弹则相安无事，否则会引发爆炸(触发ANR)

更多细节详见startService启动过程分析，http://gityuan.com/2016/03/06/start-service

#### broadcast超时机制

broadcast跟service超时机制大抵相同，对于静态注册的广播在超时检测过程需要检测SP，如下图所示。

![broadcast_anr](http://gityuan.com/images/android-anr/broadcast_anr.jpg)

图解2：

1. 客户端(App进程)向中控系统(system_server进程)发起发送广播的请求
2. 中控系统派出一名空闲的通信员(binder_1)接收该请求转交给组件管家(ActivityManager线程)
3. 组件管家执行任务(processNextBroadcast方法)的过程埋下定时炸弹
4. 组件管家通知工地(receiver所在进程)的通信员准备开始干活
5. 通讯员3号(binder_3)收到任务后转交给包工头(main主线程)，加入包工头的任务队列(MessageQueue)
6. 包工头经过一番努力干完活(完成receiver启动的生命周期)，发现当前进程还有SP正在执行写入文件的操作，便将向中控系统汇报的任务交给SP工人(queued-work-looper线程)
7. SP工人历经艰辛终于完成SP数据的持久化工作，便可以向中控系统汇报工作完成
8. 中控系统的通讯员2号(binder_2)收到包工头的完工汇报后，立刻拆除炸弹。如果在倒计时结束前拆除炸弹则相安无事，否则会引发爆炸(触发ANR)

（说明：SP从8.0开始采用名叫“queued-work-looper”的handler线程，在老版本采用newSingleThreadExecutor创建的单线程的线程池）

如果是动态广播，或者静态广播没有正在执行持久化操作的SP任务，则不需要经过“queued-work-looper”线程中转，而是直接向中控系统汇报，流程更为简单，如下图所示：

![broadcast_anr_2](http://gityuan.com/images/android-anr/broadcast_anr_2.jpg)

可见，只有XML静态注册的广播超时检测过程会考虑是否有SP尚未完成，动态广播并不受其影响。SP的apply将修改的数据项更新到内存，然后再异步同步数据到磁盘文件，因此很多地方会推荐在主线程调用采用apply方式，避免阻塞主线程，但静态广播超时检测过程需要SP全部持久化到磁盘，如果过度使用apply会增大应用ANR的概率，更多细节详见http://gityuan.com/2017/06/18/SharedPreferences

Google这样设计的初衷是针对静态广播的场景下，保障进程被杀之前一定能完成SP的数据持久化。因为在向中控系统汇报广播接收者工作执行完成前，该进程的优先级为Foreground级别，高优先级下进程不但不会被杀，而且能分配到更多的CPU时间片，加速完成SP持久化。

更多细节详见Android Broadcast广播机制分析，http://gityuan.com/2016/06/04/broadcast-receiver

#### provider超时机制

provider的超时是在provider进程首次启动的时候才会检测，当provider进程已启动的场景，再次请求provider并不会触发provider超时。

![provider_anr](http://gityuan.com/images/android-anr/provider_anr.jpg)

图解3：

1. 客户端(App进程)向中控系统(system_server进程)发起获取内容提供者的请求
2. 中控系统派出一名空闲的通信员(binder_1)接收该请求，检测到内容提供者尚未启动，则先通过zygote孵化新进程
3. 新孵化的provider进程向中控系统注册自己的存在
4. 中控系统的通信员2号接收到该信息后，向组件管家(ActivityManager线程)发送消息，埋下炸弹
5. 通信员2号通知工地(provider进程)的通信员准备开始干活
6. 通讯员4号(binder_4)收到任务后转交给包工头(main主线程)，加入包工头的任务队列(MessageQueue)
7. 包工头经过一番努力干完活(完成provider的安装工作)后向中控系统汇报工作已完成
8. 中控系统的通讯员3号(binder_3)收到包工头的完工汇报后，立刻拆除炸弹。如果在倒计时结束前拆除炸弹则相安无事，否则会引发爆炸(触发ANR)

更多细节详见理解ContentProvider原理，http://gityuan.com/2016/07/30/content-provider

#### input超时机制

input的超时检测机制跟service、broadcast、provider截然不同，为了更好的理解input过程先来介绍两个重要线程的相关工作：

- InputReader线程负责通过EventHub(监听目录/dev/input)读取输入事件，一旦监听到输入事件则放入到InputDispatcher的mInBoundQueue队列，并通知其处理该事件；
- InputDispatcher线程负责将接收到的输入事件分发给目标应用窗口，分发过程使用到3个事件队列：
  - mInBoundQueue用于记录InputReader发送过来的输入事件；
  - outBoundQueue用于记录即将分发给目标应用窗口的输入事件；
  - waitQueue用于记录已分发给目标应用，且应用尚未处理完成的输入事件；

**input的超时机制并非时间到了一定就会爆炸，而是处理后续上报事件的过程才会去检测是否该爆炸**，所以更像是扫雷的过程，具体如下图所示。

![input_anr](http://gityuan.com/images/android-anr/input_anr.jpg)

图解4：

1. InputReader线程通过EventHub监听底层上报的输入事件，一旦收到输入事件则将其放至mInBoundQueue队列，并唤醒InputDispatcher线程
2. InputDispatcher开始分发输入事件，设置埋雷的起点时间。先检测是否有正在处理的事件(mPendingEvent)，如果没有则取出mInBoundQueue队头的事件，并将其赋值给mPendingEvent，且重置ANR的timeout；否则不会从mInBoundQueue中取出事件，也不会重置timeout。然后检查窗口是否就绪(checkWindowReadyForMoreInputLocked)，**满足以下任一情况，则会进入扫雷状态(检测前一个正在处理的事件是否超时)，终止本轮事件分发**，否则继续执行步骤3。
   - 对于按键类型的输入事件，则outboundQueue或者waitQueue不为空，
   - 对于非按键的输入事件，则waitQueue不为空，且等待队头时间超时500ms
3. 当应用窗口准备就绪，则将mPendingEvent转移到outBoundQueue队列
4. 当outBoundQueue不为空，且应用管道对端连接状态正常，则将数据从outboundQueue中取出事件，放入waitQueue队列
5. InputDispatcher通过socket告知目标应用所在进程可以准备开始干活
6. App在初始化时默认已创建跟中控系统双向通信的socketpair，此时App的包工头(main线程)收到输入事件后，会层层转发到目标窗口来处理
7. 包工头完成工作后，会通过socket向中控系统汇报工作完成，则中控系统会将该事件从waitQueue队列中移除。

input超时机制为什么是扫雷，而非定时爆炸呢？是由于对于input来说即便某次事件执行时间超过timeout时长，**只要用户后续在没有再生成输入事件，则不会触发ANR**。 这里的扫雷是指当前输入系统中正在处理着某个耗时事件的前提下，后续的每一次input事件都会检测前一个正在处理的事件是否超时（进入扫雷状态），检测当前的时间距离上次输入事件分发时间点是否超过timeout时长。如果前一个输入事件，则会重置ANR的timeout，从而不会爆炸。

更多细节详见Input系统-ANR原理分析，http://gityuan.com/2017/01/01/input-anr



### ANR超时阈值

不同组件的超时阈值各有不同，关于service、broadcast、contentprovider以及input的超时阈值如下表：

![anr_timeout](http://gityuan.com/images/android-anr/anr_timeout.jpg)

#### 前台与后台服务的区别

系统对前台服务启动的超时为20s，而后台服务超时为200s，那么系统是如何区别前台还是后台服务呢？来看看ActiveServices的核心逻辑：

```Java
ComponentName startServiceLocked(...) {
    final boolean callerFg;
    if (caller != null) {
        final ProcessRecord callerApp = mAm.getRecordForAppLocked(caller);
        callerFg = callerApp.setSchedGroup != ProcessList.SCHED_GROUP_BACKGROUND;
    } else {
        callerFg = true;
    }
    ...
    ComponentName cmp = startServiceInnerLocked(smap, service, r, callerFg, addToStarting);
    return cmp;
}
```

**在startService过程根据发起方进程callerApp所属的进程调度组来决定被启动的服务是属于前台还是后台**。当发起方进程不等于ProcessList.SCHED_GROUP_BACKGROUND(后台进程组)则认为是前台服务，否则为后台服务，并标记在ServiceRecord的成员变量createdFromFg。

什么进程属于SCHED_GROUP_BACKGROUND调度组呢？进程调度组大体可分为TOP、前台、后台，进程优先级（Adj）和进程调度组（SCHED_GROUP）算法较为复杂，其对应关系可粗略理解为Adj等于0的进程属于Top进程组，Adj等于100或者200的进程属于前台进程组，Adj大于200的进程属于后台进程组。关于Adj的含义见下表，简单来说就是Adj>200的进程对用户来说基本是无感知，主要是做一些后台工作，故后台服务拥有更长的超时阈值，同时后台服务属于后台进程调度组，相比前台服务属于前台进程调度组，分配更少的CPU时间片。

![adj](http://gityuan.com/images/android-anr/adj.png)

关于细节详见解读Android进程优先级ADJ算法，http://gityuan.com/2018/05/19/android-process-adj

`前台服务准确来说，是指由处于前台进程调度组的进程发起的服务`。这跟常说的fg-service服务有所不同，fg-service是指挂有前台通知的服务。

#### 前台与后台广播超时

前台广播超时为10s，后台广播超时为60s，那么如何区分前台和后台广播呢？来看看AMS的核心逻辑：

```Java
BroadcastQueue broadcastQueueForIntent(Intent intent) {
    final boolean isFg = (intent.getFlags() & Intent.FLAG_RECEIVER_FOREGROUND) != 0;
    return (isFg) ? mFgBroadcastQueue : mBgBroadcastQueue;
}

mFgBroadcastQueue = new BroadcastQueue(this, mHandler,
        "foreground", BROADCAST_FG_TIMEOUT, false);
mBgBroadcastQueue = new BroadcastQueue(this, mHandler,
        "background", BROADCAST_BG_TIMEOUT, true);
```

**根据发送广播sendBroadcast(Intent intent)中的intent的flags是否包含FLAG_RECEIVER_FOREGROUND来决定把该广播是放入前台广播队列或者后台广播队列**，前台广播队列的超时为10s，后台广播队列的超时为60s，默认情况下广播是放入后台广播队列，除非指明加上FLAG_RECEIVER_FOREGROUND标识。

**后台广播比前台广播拥有更长的超时阈值，同时在广播分发过程遇到后台service的启动(mDelayBehindServices)会延迟分发广播，等待service的完成**，因为等待service而导致的广播ANR会被忽略掉；后台广播属于后台进程调度组，而前台广播属于前台进程调度组。简而言之，后台广播更不容易发生ANR，同时执行的速度也会更慢。

另外，只有串行处理的广播才有超时机制，因为接收者是串行处理的，前一个receiver处理慢，会影响后一个receiver；并行广播通过一个循环一次性向所有的receiver分发广播事件，所以不存在彼此影响的问题，则没有广播超时。

`前台广播准确来说，是指位于前台广播队列的广播`。

#### 前台与后台ANR

除了前台服务，前台广播，还有前台ANR可能会让你云里雾里的，来看看其中核心逻辑：

```Java
final void appNotResponding(...) {
    ...
    synchronized (mService) {
        isSilentANR = !showBackground && !isInterestingForBackgroundTraces(app);
        ...
    }
    ...
    File tracesFile = ActivityManagerService.dumpStackTraces(
            true, firstPids,
            (isSilentANR) ? null : processCpuTracker,
            (isSilentANR) ? null : lastPids,
            nativePids);

    synchronized (mService) {
        if (isSilentANR) {
            app.kill("bg anr", true);
            return;
        }
        ...

        //弹出ANR选择的对话框
        Message msg = Message.obtain();
        msg.what = ActivityManagerService.SHOW_NOT_RESPONDING_UI_MSG;
        msg.obj = new AppNotRespondingDialog.Data(app, activity, aboveSystem);
        mService.mUiHandler.sendMessage(msg);
    }
}
```

决定是前台或者后台ANR取决于该应用发生ANR时对用户是否可感知，比如拥有当前前台可见的activity的进程，或者拥有前台通知的fg-service的进程，这些是用户可感知的场景，发生ANR对用户体验影响比较大，故需要弹框让用户决定是否退出还是等待，如果直接杀掉这类应用会给用户造成莫名其妙的闪退。

后台ANR相比前台ANR，只抓取发生无响应进程的trace，也不会收集CPU信息，并且会在后台直接杀掉该无响应的进程，不会弹框提示用户。

`前台ANR准确来说，是指对用户可感知的进程发生的ANR`。

### ANR爆炸现场

对于service、broadcast、provider、input**发生ANR后，中控系统会马上去抓取现场的信息，用于调试分析**。收集的信息包括如下：

- 将am_anr信息输出到EventLog，也就是说ANR触发的时间点最接近的就是EventLog中输出的`am_anr`信息
- 收集以下重要进程的各个线程调用栈trace信息，保存在**data/anr/traces.txt**文件
  - 当前**发生ANR的进程**，system_server进程以及所有persistent进程
  - audioserver, cameraserver, mediaserver, surfaceflinger等重要的**native进程**
  - **CPU使用率排名前5**的进程
- 将发生ANR的**reason以及CPU使用情况**信息输出到 **main log**
- 将traces文件和CPU使用情况信息保存到dropbox，即**data/system/dropbox**目录
- 对用户可感知的进程则弹出ANR对话框告知用户，对用户不可感知的进程发生ANR则直接杀掉

**整个ANR信息收集过程比较耗时**，其中抓取进程的trace信息，每抓取一个等待200ms，可见persistent越多，等待时间越长。关于抓取trace命令，对于Java进程可通过在adb shell环境下执行kill -3 [pid]可抓取相应pid的调用栈；对于Native进程在adb shell环境下执行debuggerd -b [pid]可抓取相应pid的调用栈。对于ANR问题发生后的蛛丝马迹(trace)在traces.txt和dropbox目录中保存记录。更多细节详见理解Android ANR的信息收集过程，http://gityuan.com/2016/12/02/app-not-response。

有了现场信息，可以调试分析，先定位发生ANR时间点，然后查看trace信息，接着分析是否有耗时的message、binder调用，锁的竞争，CPU资源的抢占，以及结合具体场景的上下文来分析，调试手段就需要针对前面说到的message、binder、锁等资源从系统角度细化更多debug信息，这里不再展开，后续再以ANR案例来讲解。

作为应用开发者

* 应让主线程尽量只做UI相关的操作，避免耗时操作，比如过度复杂的UI绘制，网络操作，文件IO操作；
* 避免主线程跟工作线程发生锁的竞争，
* 减少系统耗时binder的调用，
* 谨慎使用sharePreference，
* 注意主线程执行provider query操作。
* 简而言之，尽可能减少主线程的负载，让其空闲待命，以期可随时响应用户的操作。



#### 回答

最后，来回答文章开头的提问，有哪些路径会引发ANR? 

* 答应是从埋下定时炸弹到拆炸弹之间的任何一个或多个路径执行慢都会导致ANR（以service为例）：
  * 可以是service的生命周期的回调方法(比如onStartCommand)执行慢，
  * 可以是主线程的消息队列存在其他耗时消息让service回调方法迟迟得不到执行，
  * 可以是SP操作执行慢，
  * 可以是system_server进程的binder线程繁忙而导致没有及时收到拆炸弹的指令。
  * 另外ActivityManager线程也可能阻塞，出现的现象就是前台服务执行时间有可能超过10s，但并不会出现ANR。

发生ANR时从trace来看主线程却处于空闲状态或者停留在非耗时代码的原因有哪些？

* 可以是抓取trace过于耗时而错过现场，
* 可以是主线程消息队列堆积大量消息而最后抓取快照一刻只是瞬时状态，
* 可以是广播的“queued-work-looper”一直在处理SP操作。