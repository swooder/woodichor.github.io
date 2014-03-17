---
layout: post
title: "Android Asynchronous HandlerThread"
date: 2014-03-17 23:06:29 +0800
comments: true
categories: Android
---
#### Looper
Looper是用于给一个线程添加一个消息队列(MessageQueue)，并且循环等待，当有消息时会唤起线程来处理消息的一个工具，直到线程结束为止。通常情况下不会用到Looper，因为对于Activity，Service等系统组件，Frameworks已经为我们初始化好了线程(俗称的UI线程或主线程)，在其内含有一个Looper，和由Looper创建的消息队列，所以主线程会一直运行，处理用户事件，直到某些事件(BACK）退出。
如果，我们需要新建一个线程，并且这个线程要能够循环处理其他线程发来的消息事件，或者需要长期与其他线程进行复杂的交互，这时就需要用到Looper来给线程建立消息队列.
#####使用Looper:
```java  Looper http://www.shaojie.name
  class LooperThread extends Thread {
      public Handler mHandler;

      public void run() {
          Looper.prepare();

          mHandler = new Handler() {
              public void handleMessage(Message msg) {
                  // process incoming messages here
              }
          };

          Looper.loop();
      }
  }
```

这样你的线程就具有了消息处理机制了，在Handler中进行消息处理.
Looper最主要的有四个:
```
 public static prepare();
 public static myLooper();
 public static loop();
 public void quit();
```
使用方法如下：

1. 在每个线程的run()方法中的最开始调用Looper.prepare()，这是为线程初始化消息队列。
2. 之后调用Looper.myLooper()获取此Looper对象的引用。这不是必须的，但是如果你需要保存Looper对象的话，一定要在prepare()之后，否则调用在此对象上的方法不一定有效果，如looper.quit()就不会退出。
3. 在run()方法中添加Handler来处理消息
4. 添加Looper.loop()调用，这是让线程的消息队列开始运行，可以接收消息了。
5. 在想要退出消息循环时，调用Looper.quit()注意，这个方法是要在对象上面调用，很明显，用对象的意思就是要退出具体哪个Looper。如果run()中无其他操作，线程也将终止运行。

#### Handler
Handler是用于操作线程内部的消息队列的类。Looper是用于给线程创建消息队列用的，也就是说Looper可以让消息队列(MessageQueue)附属在线程之内，并让消息队列循环起来，接收并处理消息。但，我们并不直接的操作消息队列，而是用Handler来操作消息队列，给消息队列发送消息，和从消息队列中取出消息并处理。这就是Handler的职责。

Handler主要有二个用途，一个是用于线程内部消息循环； 另外一个就是用于线程间通讯，主要方法：
```
Handler.sendEmptyMessageDelayed(int msgid, long after);
Handler.sendMessageDelayed(Message msg, long after);
Handler.postDelayed(Runnable task, long after);
Handler.sendMessageAtTime(Message msg, long timeMillis);
Handler.sendEmptyMessageAtTime(int id, long timeMiilis);
Handler.postAtTime(Runnable task, long timeMillis);
``` 
需要注意一点的是，线程内部消息循环并不是并发处理，也就是所有的消息都是在Handler所属的线程内处理的，所以虽然你用post(Runnable r)，发给MessageQueue一个Runnable，但这并不会创建新的线程来执行，处理此消息时仅是调用r.run()。（想要另起线程执行，必须把Runnable放到一个Thread中）。

####Handler 
一般会使用Handler handler = new Handler(){...}创建Handler。这样创建的handler是在主线程即UI线程下的Handler，
即这个Handler是与UI线程下的默认Looper绑定的。Looper是用于实现消息队列和消息循环机制的。
因此，如果是默认创建Handler那么如果线程是做一些耗时操作如网络获取数据等操作，这样创建Handler是不行的.
Android API提供了HandlerThread来创建线程。官网的解释是：Handy class for starting a new thread that has a looper.
The looper can then be used to create handler classes. Note that start() must still be called.

HandlerThread实际上就一个Thread，只不过它比普通的Thread多了一个Looper。
```
public class HandlerThread extends Thread {
 private int mPriority;
 private int mTid = -1;
 private Looper mLooper;
 ....
 public void run(){
 	 mTid = Process.myTid();
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            Process.setThreadPriority(mPriority);
            notifyAll();
        }
        onLooperPrepared();
        Looper.loop();
        mTid = -1;
 }
 ....
 }
```

创建HandlerThread时要把它启动了，即调用start()方法。然后创建Handler时将HandlerThread中的looper对象传入。
HandlerThread thread = new HandlerThread("MyHandlerThread");
thread.start();
mHandler = new Handler(thread.getLooper());
mHandler.post(new Runnable(){...});
那么这个Handler对象就是与HandlerThread这个线程绑定了（这时就不再是与UI线程绑定了，这样它处理耗时操作将不会阻塞UI。

```
private Handler mHandler;
private HandlerThread mHandlerThread;

private boolean mRunning;

private Button btn;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    btn = (Button) findViewById(R.id.btn);
    btn.setOnClickListener(this);

    mHandlerThread = new HandlerThread("Test", 5);
    mHandlerThread.start();
    mHandler = new Handler(mHandlerThread.getLooper());
}

private Runnable mRunnable = new Runnable() {

    @Override
    public void run() {
        while (mRunning) {
            Log.d("MainActivity", "test HandlerThread...");
            try {
                Thread.sleep(200);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

    }
};

@Override
public void onClick(View v) {
    switch(v.getId()) {
    case R.id.btn :
        mHandler.post(mRunnable);
        break;
    default :
        break;
    }

}

@Override
protected void onDestroy() {
    mRunning = false;
    mHandler.removeCallbacks(mRunnable);
    super.onDestroy();
}

@Override
protected void onResume() {
    mRunning = true;
    super.onResume();
}

@Override
protected void onStop() {
    mRunning = false;
    super.onStop();
}
```





