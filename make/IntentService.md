# IntentService完全解析，带你从源码角度彻底理解(全)
上篇中有了源代码以后，我们就可以对某些关键的**源代码进行分析了，看看它们的实现原理**，比如像IntentService，HandlerThread，AMS，PMS分析，在面试的过程中，也会经常被问到某某的实现原理，这里**源码分析篇**会完成以下内容的介绍
![](https://github.com/qydq/anopen/blob/master/screen/mind_android_kernel_analysis.png?raw=true)

>说一点面试经验ba，其实根据面试官的提问，我们大概也能评估出面试官是什么水平，有的面试官水平并不一定就高于你，保持谦逊的态度，根据面试官的水平来进行回答，并不是生搬硬套的内容，对于一些知识点一定是要先理解其本质

通过本篇你会了解到**IntentService的一个实现原理**，Service和IntentService的使用，以及它们的区别，这里本神就把Service也包含进去了，还有一些注意事项，123等等，当然其中比较重要的就是```HandlerThread```.

本来想单独一篇内容，但我也不是什么作家，我只是告诉自己的一定要控制篇幅和内容的质量，篇数一定要少而精，内容一定不能重复，站在前人的肩膀上成长，有转载和引用要适当给出参考链接，尊重原文出处。还有就是文章的归档也很重要，```芳芳更重要```

> 文章归档：[ word 20180106] -  `ina系列Android树`4章节.markdown
---
## 带你从源码角度彻底理解IntentService

官方提供了一个IntentService，介绍IntentSerrvice之前必须先介绍Service，Android Service是运行在后台的代码，不能与用户交互，```可以运行在自己的进程，也可以运行在其他应用程序进程的上下文里```，需要通过某一个Activity或者Context对象来调用。下面按照一定顺序由**浅到深介绍IntentService**
* .Service和IntentService的区别
* .android进程保活大全(腾讯-张兴华前辈)
* .IntentService源码分析
* .android 8.0启动后台Service易错点总结

**版权声明CopyRight：**

> **本内容作者：sunst，转载或引用请**[标明出处](https://www.zhihu.com/people/qydq)**，违者追究法律责任！！！**

本篇为所有知识梳理的标准格式，都应参考该格式

## 一：Service和IntentService的区别

### 1.Service简单回顾
Service不多说，它是长期运行在后台的应用程序组件，不是一个单独的进程，它和应用程序在同一个进程中，Service 也不是一个线程，它和线程没有任何关系，所以它不能直接处理耗时操作。如果直接把耗时操作放在 Service 的 onStartCommand() 中，很容易引起 ANR ，所以
> 如果在Service执行耗时的操作需要启动一个新线程来执行。

#### Service有两个启动方法，分别是

>Context.startService()
>Context.bindService()

-   startService(): 调用者与服务之间没有关连，调用者退出后Service仍然存在。
-   bindService(): 调用者与服务绑定在了一起，调用者一旦退出，Service也随即终止。

为控制篇幅不再过多介绍，更详细的可以参考：[推荐1：Android服务两种启动方式的区别](http://www.jianshu.com/p/2fb6eb14fdec)。

### 2.IntentService
关于IntentService的介绍在源代码可以看到这样的注释
>IntentService is a base class for Services that handle asynchronous requests (expressed as Intents) on demand. Clients send requests through startService(Intent) calls; the service is started as needed, handles each Intent in turn using a worker thread, and stops itself when it runs out of work.

IntentService是继承于Service的子类，它可以处理异步处理请求，**Client使用startService(Intent)来启动Service**。在它内部有一个工作线程来处理耗时操作，当它处理完所有请求的时候会停止它自身，不需要我们手动控制。另外，可以启动IntentService多次，而每一个耗时操作会以工作队列的方式在IntentService 的 ```onHandleIntent```回调方法中执行，并且，每次只会执行一个工作线程，执行完第一个再执行第二个，以此类推。

而且，所有请求都在一个单线程中，不会阻塞应用程序的主线程（UI Thread），同一时间只处理一个请求。

**那么，用 IntentService 有什么好处呢？**

> 1.省去了在 Service 中手动开线程的麻烦
> 2.当操作完成时，我们不用手动停止Service

接下来让我们来看看如何使用，写一个Demo来模拟两个耗时操作。

### 3.Service和IntentService耗时操作比较

先分别在 Service 和 IntentService 里面处理耗时，看看是什么结果。

**（1）.首先创建一个 Service 服务，在其 onStartCommand() 中执行一个20s的耗时。**
```java
public class MyService extends Service {  
    String TAG = "MyService";  
    @Override  
  public IBinder onBind(Intent intent) {  
        return null;  
    }  
    @Override  
  public int onStartCommand(Intent intent, int flags, int startId) {  
        Log.d(TAG, "-->start");  
        try {  
            Thread.sleep(20000);  
        } catch (InterruptedException e) {  
        }  
        Log.d(TAG, "-->end");  
        return super.onStartCommand(intent, flags, startId);  
    }  
}
```
**Step2：在activity中启动一个Service**
```java
Intent intent =  new  Intent(mContext, MyService.class);  startService(intent);
```
不用猜也知道结果会怎么样

> 出现：ANR

**（2）.使用IntentService，在其onHandleIntent() 中再执行一个20s的耗时。**
```java
public class MyIntentService extends IntentService {  
    String TAG = "MyIntentService";  
    public MyIntentService() {  
        super("sunst");  
    }  
    @Override  
  protected void onHandleIntent(Intent intent) {  
        Log.d(TAG, "-->start");  
        try {  
            Thread.sleep(20000);  
        } catch (InterruptedException e) {  
        }  
        Log.d(TAG, "-->end");  
    }
```
IntentService 默认实现了 OnBind()，返回值为 null，IntentService 必须实现 MyIntentService() 构造方法和 onHandleIntent(Intent intent)。

**注意：**
> IntentService 的构造函数一定是参数为空的构造函数，然后再在其中调用 super(“name”) 这种形式的构造函数。 因为 Service 的实例化是系统来完成的，而且系统是用参数为空的构造函数来实例化Service的。

**同样，Step2：在activity中启动MyIntentService**
```java
Intent intent =  new  Intent(mContext, MyIntentService.class);  
startService(intent);
```
> 启动 IntentService，果然没有发生ANR。

**（3）.验证IntentService异步处理的能力**
这里我针对IntentService，在activity中写一个循环，不断的启动IntentService
```java
for(int i = 0; i < 20; i++) {
	Intent intent = new Intent(mContext, MyIntentService.class);
	startService(intent);
 }  
```
直接结果是：IntentService中，循环打印了20次，[-->start ：-->end]这样的内容，**为此得出一个结论**

> IntentService采用单独的线程每次只从队列中拿出一个请求进行处理

### 4.Service和IntentService中onStartCommand返回值比较

Android中调用startService(Intent intent)会调用该Service对象的onStartCommand(Intent intent, int flags, int startId)方法，然后在onStartCommand方法中做一些处理。这个函数有一个int的返回值，根据官方文档说明，返回值有四种情况，下面就对这四种返回值做一一介绍Service类中```onStartCommand```返回值

**1⃣️START_STICKY**

“粘性的”。服务被异常kill掉，保留service的状态为开始状态，但不保留递送的intent对象。随后系统会尝试重新创建service，由于服务状态为开始状态，所以创建服务后一定会调用onStartCommand(Intent,int,int)方法。如果在此期间没有任何启动命令被传递到service，那么参数Intent将为null。

**2⃣️START_NOT_STICKY**

“非粘性的”。使用这个返回值时，如果在执行完onStartCommand后，服务被异常kill掉，系统不会自动重启该服务。

**3⃣️START_REDELIVER_INTENT**

重传Intent。使用这个返回值时，如果在执行完onStartCommand后，服务被异常kill掉，系统会自动重启该服务，并将Intent的值传入。

**4⃣️START_STICKY_COMPATIBILITY**

START_STICKY的兼容版本，但不保证服务被kill后一定能重启。

#### （1）.Service系统默认策略

Service的onStartCommand策略
```java
public int onStartCommand(Intent intent, int flags, int startId) {  
    onStart(intent, startId);  
    return mStartCompatibility ? START_STICKY_COMPATIBILITY : START_STICKY;  
}
```
可见，默认的策略是START_STICKY，支持服务意外终止重新创建的。

#### （2）.IntentService的onStartCommand策略

IntentService不应该重新实现onStartCommand，而是去复写 onHandleIntent .
```java
@Override
  public int onStartCommand(Intent intent, int flags, int startId) {
      onStart(intent, startId);
      return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
  }
  public void setIntentRedelivery(boolean enabled) {
         mRedelivery = enabled;
   }
```
可见， IntentService 默认只支持两种返回值 START_REDELIVER_INTENT 或者 START_NOT_STICKY ，并且由 setIntentRedelivery 方法决定，默认是 START_NOT_STICKY ,不重新创建。
## 二：android进程保活大全(腾讯-张兴华前辈)
Android 进程拉活包括两个层面：

1.  提供进程优先级，降低进程被杀死的概率
2.  在进程被杀死后，进行拉活

进程的保活可以参考腾讯，[推荐1：张兴华的这篇文章click](https://segmentfault.com/a/1190000006251859)，介绍的非常详细，大概就是启动1个像素的Activity，以及利用Service来做拉活的工作

进程的保活涉及到前后台Service，另外一篇，[推荐2：Android四大组件Service之前台进程](https://blog.csdn.net/tanmx219/article/details/81255504)

>这里突然想到广播，那说一下吧，如果在Service里面拉起一个BroadCastRecever的话，一定要对广播设置一个flag，intent.setFlag（）,否则activity中有可能收不到广播。

## 三：IntentService源码分析

IntentService 为什么可以处理耗时任务？ 应该从源码上面来分析,接下来就来看看IntentService的源码：
```java
public abstract class IntentService extends Service {
    private volatile Looper mServiceLooper;
    private volatile ServiceHandler mServiceHandler;
    private String mName;
    private boolean mRedelivery;

    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }
        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);
        }
    }
    public IntentService(String name) {
        super();
        mName = name;
    }
  	//设置Intent是否重投递，如果重投递，在onHandleIntent方法返回前，
  	//如果进程死了，会重启进程，重新分发Intent
  	//但是只会分发最近的一个Intent
    public void setIntentRedelivery(boolean enabled) {
        mRedelivery = enabled;
    }
    @Override
    public void onCreate() {
        super.onCreate();
        //通过实例化HandlerThreade新建线程&启动
        //故，使用IntentService时，不需要额外新建线程。
        //HandlerThread继承自Thread，内部封装了Looper
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        thread.start();
		//获取工程线程的Looper&维护自己的工作队列
        mServiceLooper = thread.getLooper();
		//新建ServiceHandler，绑定上面的Looper
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }
    @Override
    public void onStart(@Nullable Intent intent, int startId) {
    	//获取ServiceHandler消息的引用
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        //发送消息，添加到消息队列
        mServiceHandler.sendMessage(msg);
    }
    @Override
    public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
    	//调用omnStart()方法
        onStart(intent, startId);
        return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
    }

    @Override
    public void onDestroy() {
    	//Looper停止
        mServiceLooper.quit();
    }

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @WorkerThread
    protected abstract void onHandleIntent(@Nullable Intent intent);
}
//Service.java
 public final void stopSelf(int startId) {
        if (mActivityManager == null) {
            return;
        }
        try {
            mActivityManager.stopServiceToken(
                    new ComponentName(this, mClassName), mToken, startId);
        } catch (RemoteException ex) {
        }
    }
```

IntentService 是直接继承与 Service的，继承Service后,它的代码一共就100多行

### 1.onCreate()

onCreate方法中新建了一个HandlerThread实例。  
>HandlerThread 是一个Thread的子类，HandlerThread内部有点线我们的UI线程，内部一个Looper loop循环一直轮询获得消息，然后处理消息。
 
内部有一个Handler子类ServiceHandler，它的Looper用的是这个HandlerThread 的Looper,IntentSerivce 在onStart()通过发送Message,ServiceHandler在处理Message调用的是 onHandleIntent。 

简单的说一个IntentService,内部就创建了一个线程，通过Android提供的 Handler Message Looper,这些消息处理的类 构成了一个消息处理的模型。所以IntentService 的onHandleIntent 这个方法其实是在IntentService 中开辟的一个子线程中处理的

### 2.onStart()
Service在执行任务时会调用onStartCommand方法，看下onStartCommand，发现会调用onStart方法
```java
@Override
public void onStart(@Nullable Intent intent, int startId) {
Message msg = mServiceHandler.obtainMessage();
msg.arg1 = startId;
msg.obj = intent;
mServiceHandler.sendMessage(msg);
}
```
onStart方法里会调用handler的sendMeasage方法，sendMessage后就会调用ServieHanlder里的handMessage方法，handleMessage方法调用了onHandleIntent方法，这个方法是抽象方法，需要在子类中重写，耗时的任务的处理就写在这个方法里，处理完任务调用stopSelf，即关闭Service。

## 四：android 8.0启动后台Service易错点总结

### 1.不建议通过bindService()启动IntentService
如果采用bindService()启动IntentService的声明周期：
onCreate() ->> onBind() ->> onunbind()->> onDestory()
即，并不会调用onStart或onStartCommand，所以不会将消息发送给消息队列，那么onHandleIntent将不会调用。即无法实现多线程的操作。

### 2.若服务停止，则会清除消息队列中的消息，后续的事件不执行。再调用了startService，将重新开启一个新的IntentService。

### 3.[Android 8.0 启动后台service 出错 IllegalStateException: Not allowed to start service Intent](https://blog.csdn.net/kdsde/article/details/82143866)

### 4.对于Service在binderService时可能遇到如下这样的错误
```
E/ActivityThread: Activity com.example.image.all_samples.Main2Activity has leaked ServiceConnection com.example.image.all_samples.Main2Activity$5@d124870 that was originally bound here  
android.app.ServiceConnectionLeaked: Activity com.example.image.all_samples.Main2Activity has leaked ServiceConnection com.example.image.all_samples.Main2Activity$5@d124870 that was originally bound here  
at android.app.LoadedApk$ServiceDispatcher.<init>(LoadedApk.java:1369)  
at android.app.LoadedApk.getServiceDispatcher(LoadedApk.java:1264)  
at android.app.ContextImpl.bindServiceCommon(ContextImpl.java:1454)  
at android.app.ContextImpl.bindService(ContextImpl.java:1426)  
at android.content.ContextWrapper.bindService(ContextWrapper.java:636)  
at com.example.image.all_samples.Main2Activity$3.onClick(Main2Activity.java:53)  
at android.view.View.performClick(View.java:5724)  
at android.view.View$PerformClick.run(View.java:22572)  
at android.os.Handler.handleCallback(Handler.java:751)  
at android.os.Handler.dispatchMessage(Handler.java:95)  
at android.os.Looper.loop(Looper.java:154)  
at android.app.ActivityThread.main(ActivityThread.java:6259)  
at java.lang.reflect.Method.invoke(Native Method)  
at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:913)  
at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:803)  
```
**异常原因：**  
>一般为在activity退出时未调用unbindService导致，只需要添加unbindService接口即可。  
  
进一步思考：android系统是如何检测到泄漏的呢。  
很容想到bindService时，是通过当前activity的context进行bind的，所以service的生命周期应该是同改activity关联，进一步验证，bindService时通过application的context进行bind，退出当前activity时，不会报上面的异常错误，基本可以证明service的生命周期同bindService时依赖的context相关，

>所以如果需要一个不依赖于具体activity生命周期的service，用application的context进行bind即可。
---
以上便是IntentService完全解析，带你从源码角度彻底理解(全)的全部内容。

请尊重劳动成果，注意文中**版权声明**，**Android专栏不定时更新，可以[点击关注我知乎](https://www.zhihu.com/people/qydq/activities)。也可以同时关注[人工智能专栏](https://zhuanlan.zhihu.com/sstai)，本内容作者**`sunst`**，有问题请沟通`qyddai@gmail.com`

> 作者：sunst 2019-08-18 19:59