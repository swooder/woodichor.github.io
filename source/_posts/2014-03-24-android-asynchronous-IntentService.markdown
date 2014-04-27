---
layout: post
title: "Android Asynchrous IntentService"
date: 2014-03-24 23:05:38 +0800
comments: true
categories: Android
---
####Service
######Service种类

|类别 | 区别  |  优点   |缺点   |应用   |
|---|---|---|---|
| 本地服务（Local)  | 该服务依附在主进程上  | 服务依附在主进程上而不是独立的进程，这样在一定程度上节约了资源，另外Local服务因为是在同一进程因此不需要IPC，也不需要AIDL。相应bindService会方便很多。  | 主进程被Kill后，服务便会终止|非常常见的应用如：音乐播放服务，Push服务。  |   
| 远程服务（Remote）  | 该服务是独立的进程  | 服务为独立的进程，对应进程名格式为所在包名加上你指定的android:process字符串。由于是独立的进程，因此在Activity所在进程被Kill的时候，该服务依然在运行，不受其他进程影响，有利于为多个进程提供服务具有较高的灵活性  | 该服务是独立的进程，会占用一定资源，并且使用AIDL进行IPC稍微麻烦一点| 一些提供系统服务的Service，这种Service是常驻的  |   

#####service运行分类
|类别 | 区别 |应用|
|---|---|---|
|前台服务|会在通知一栏显示 ONGOING 的 Notification|当服务被终止的时候，通知一栏的 Notification 也会消失，这样对于用户有一定的通知作用。常见的如音乐播放服务|
|后台服务|默认的服务即为后台服务，即不会在通知一栏显示 ONGOING 的 Notification|当服务被终止的时候，用户是看不到效果的。某些不需要运行或终止提示的服务，如天气更新，日期同步，邮件同步等|

#####Service 与 Thread的区别
1. Thread：Thread是程序执行的最小单元，它是分配CPU的基本单位。可以用 Thread 来执行一些异步的操作。
2. Service：Service 是Android的一种机制，当它运行的时候如果是Local Service，那么对应的 Service 是运行在主进程的 main 线程上的。如：onCreate，onStart 这些函数在被系统调用的时候都是在主进程的 main 线程上运行的。如果是Remote Service，那么对应的 Service 则是运行在独立进程的 main 线程上。

那么我们为什么要用 Service 呢？其实这跟 android 的系统机制有关，我们先拿 Thread 来说。Thread 的运行是独立于 Activity 的，也就是说当一个 Activity 被 finish 之后，如果你没有主动停止 Thread 或者 Thread 里的 run 方法没有执行完毕的话，Thread 也会一直执行。因此这里会出现一个问题：当 Activity 被 finish 之后，你不再持有该 Thread 的引用。另一方面，你没有办法在不同的 Activity 中对同一 Thread 进行控制。


#####Service生命周期
1. 三个阶段：创建-开始-销毁
2. Oncreate();Service 声明周期的开始，完成Service的初始化工作
3. OnStartCommand(); 服务声明周期开始，没有对应的停止函数
4. OnDestory(); 声明周期的结束，释放Service所占用的资源

#####Service启动方式
1. Conetext.startService()
2. Conetext.bindService();

####IntentService
IntentService继承与Service，它最大的特点是对服务请求逐个进行处理。当我们要提供的服务不需要同时处理多个请求的时候，可以选择继承IntentService。

#####使用IntentService有几个需要注意的地方：
1. 它不能直接与用户交互，如果想把数据传递给用户界面，需要把值先传递给Activity
2. 一个intentService可以处理多个任务，只不过是一个接着一个的顺序来处理的,后面的任务需要在前一个完成后才能执行
3. 运行在IntetService中的操作是不能被打断的

###### IntentService的特点
1.  它创建了一个独立的工作线程来处理所有的通过onStartCommand()传递给服务的intents。

2.  创建了一个工作队列，来逐个发送intent给onHandleIntent()。

3.   不需要主动调用stopSelft()来结束服务。因为，在所有的intent被处理完后，系统会自动关闭服务。

4.   默认实现的onBind()返回null

5.  默认实现的onStartCommand()的目的是将intent插入到工作队列中。

######源码分析
```
java Interservice www.shaojie.name
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
```
从源码可以分析出:
IntentService 实际上是Looper,Handler,Service 的集合体,他不仅有服务的功能,还有处理和循环消息的功能

```     
@Override 
public void onCreate() { 
        super.onCreate(); 

        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]"); 
        thread.start(); 

        mServiceLooper = thread.getLooper(); 
        mServiceHandler = new ServiceHandler(mServiceLooper); 
        }
```
IntentService创建时就会创建Handler线程(HandlerThread)并且启动,然后再得到当前线程的Looper对象来初始化IntentService的mServiceLooper,接着创建mServicehandler对象.

```
@Override 
        public void onStart(Intent intent, int startId) { 
                Message msg = mServiceHandler.obtainMessage(); 
                msg.arg1 = startId; 
                msg.obj = intent; 

                mServiceHandler.sendMessage(msg); 
        }
```
当你启动IntentService的时候,就会产生一条附带startId和Intent的Message并发送到MessageQueue中,接下来Looper发现MessageQueue中有Message的时候,就会停止Handler处理消息.

```
 @Override 
        public void handleMessage(Message msg) { 
                        onHandleIntent((Intent)msg.obj); 
                        stopSelf(msg.arg1); 
        }
```
接着调用 onHandleIntent((Intent)msg.obj),这是一个抽象的方法,其实就是我们要重写实现的方法,我们可以在这个方法里面处理我们的工作.当任务完成时就会调用stopSelf(msg.arg1)这个方法来结束指定的工作

```
@Override 
        public void onDestroy() { 
                mServiceLooper.quit(); 
        }
```
当所有的工作执行完后:就会执行onDestroy方法,服务结束后调用这个方法 mServiceLooper.quit()使looper停下来.

 

