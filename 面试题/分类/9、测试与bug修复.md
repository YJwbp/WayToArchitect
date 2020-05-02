### 32、单元测试有没有做过，说说熟悉的单元测试框架？

首先，Android测试主要分为三个方面：

- 单元测试（Junit4、Mockito、PowerMockito、Robolectric）
- UI测试（Espresso、UI Automator）
- 压力测试（Monkey）

WanAndroid项目和XXX项目中使用用到了单元测试和部分自动化UI测试，其中单元测试使用的是Junit4+Mockito+PowerMockito+Robolectric。下面我分别简单介绍下这些测试框架：

1、Junit4：

使用@Test注解指定一个方法为一个测试方法，除此之外，还有如下常用注解@BeforeClass->@Before->@Test->@After->@AfterClass以及@Ignore。

Junit4的主要测试方法就是断言，即assertEquals()方法。然后，你可以通过实现TestRule接口的方式重写apply()方法去自定义Junit Rule，这样就可以在执行测试方法的前后做一些通用的初始化或释放资源等工作，接着在想要的测试类中使用@Rule注解声明使用JsonChaoRule即可。（注意被@Rule注解的变量必须是final的。最后，我们直接运行对应的单元测试方法或类，如果你想要一键运行项目中所有的单元测试类，直接点击运行Gradle Projects下的app/Tasks/verification/test即可，它会在module下的build/reports/tests/下生成对应的index.html报告。

Junit4它的优点是速度快，支持代码覆盖率如jacoco等代码质量的检测工具。缺点就是无法单独对Android UI，一些类进行操作，与原生Java有一些差异。

2、Mockito：

可以使用mock()方法模拟各种各样的对象，以替代真正的对象做出希望的响应。除此之外，它还有很多验证方法调用的方式如Mockit.when(调用方法).thenReturn(验证的返回值)、verfiy(模拟对象).验证方法等等。

这里有一点要补充下：简单的测试会使整体的代码更简洁，更可读、更可维护。如果你不能把测试写的很简单，那么请在测试时重构你的代码。

最后，对于Mockito来说，它的优点是有各种各样的方式去验证"模仿对象"的互动或验证发生的某些行为。而它的缺点就是不支持mock匿名类、final类、static方法private方法。

3、PowerMockito：

因此，为了解决Mockito的缺陷，PoweMockito出现了，它扩展了Mockito，支持mock匿名类、final类、static方法、private方法。只要使用它提供的api如PowerMockito.mockStatic()去mock含静态方法或字段的类，PowerMockito.suppress(PowerMockito.method(类.class， 方法名)即可。

4、Robolectric

前面3种我们说的都是Java相关的单元测试方法，如果想在Java单元测试里面进行Android单元测试，还得使用Robolectric，它提供了一套能运行在JVM的Android代码。它提供了一系列类似ShadowToast.getLatestToast()、ShadowApplication.getInstance()这种方式来获取Android平台对应的对象。可以看到它的优点就是支持大部分Android平台依赖类的底层引用与模拟。缺点就是在异步测试的情况下有些问题，这是可以结合Mockito来将异步转为同步即可解决。

最后，自动化UI测试项目中我使用的是Expresso，它提供了一系列类似onView().check().perform()的方式来实现点击、滑动、检测页面显示等自动化的UI测试效果，这里在我的WanAndroid项目下的BasePageTest基类里面封装了一系列通用的方法，有兴趣可以去看看。



### 22、CrashHandler实现原理？

    获取app crash的信息保存在本地然后在下一次打开app的时候发送到服务器。

   

#### 1、什么是ANR 如何避免它？

答：在Android上，如果你的应用程序有一段时间响应不够灵敏，系统会向用户显示一个对话框，这个对话框称作应
用程序无响应（ANR：Application NotResponding）对话框。
用户可以选择让程序继续运行，但是，他们在使用你的
应用程序时，并不希望每次都要处理这个对话框。因此
，在程序里对响应性能的设计很重要这样，这样系统就不会显
示ANR给用户。

不同的组件发生ANR的时间不一样，Activity是5秒，BroadCastReceiver是10秒，Service是20秒（均为前台）。

如果开发机器上出现问题，我们可以通过查看/data/anr/traces.txt即可，最新的ANR信息在最开始部分。

- 主线程被IO操作（从4.0之后网络IO不允许在主线程中）阻塞。
- 主线程中存在耗时的计算
- 主线程中错误的操作，比如Thread.wait或者Thread.sleep等 Android系统会监控程序的响应状况，一旦出现下面两种情况，则弹出ANR对话框
- 应用在5秒内未响应用户的输入事件（如按键或者触摸）
- BroadcastReceiver未在10秒内完成相关的处理
- Service在特定的时间内无法处理完成 20秒

修正：

1、使用AsyncTask处理耗时IO操作。

2、使用Thread或者HandlerThread时，调用Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND)设置优先级，否则仍然会降低程序响应，因为默认Thread的优先级和主线程相同。

3、使用Handler处理工作线程结果，而不是使用Thread.wait()或者Thread.sleep()来阻塞主线程。

4、Activity的onCreate和onResume回调中尽量避免耗时的代码。
BroadcastReceiver中onReceive代码也要尽量减少耗时，建议使用IntentService处理。


##### 解决方案：

将所有耗时操作，比如访问网络，Socket通信，查询大
量SQL 语句，复杂逻辑计算等都放在子线程中去，然
后通过handler.sendMessage、runonUIThread、AsyncTask、RxJava等方式更新UI。无论如何都要确保用户界面的流畅
度。如果耗时操作需要让用户等待，那么可以在界面上显示度条。

[深入回答](http://mp.weixin.qq.com/s?__biz=MzIwMTAzMTMxMg==&mid=2649493643&idx=1&sn=34b51d1f61bd2ecaa8fd0a2d39c4d1d1&chksm=8eec9b74b99b126246acc4547597dfe55c836b8f689b2d1a65bdf1ee2054ced2fc070bfa2678&mpshare=1&scene=24&srcid=0116vzNfMMv2dLizhAT8mEYq#rd)


#### 