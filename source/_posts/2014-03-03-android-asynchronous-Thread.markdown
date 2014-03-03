---
layout: post
title: "Android Asynchronous Thread"
date: 2014-03-03 22:32:57 +0800
comments: true
categories: Android
---

### 线程概述
线程是一个程序的多个执行路径，执行调度的单位，依托于进程存在。 线程不仅可以共享进程的内存，而且还拥有一个属于自己的内存空间，这段内存空间也叫做线程栈，是在建立线程时由系统分配的，主要用来保存线程内部所使用的数据，如线程执行函数中所定义的变量。
###线程定义
####继承java.lang.Thread类
``` java thread http://www.shaojie.name
public class MyThread extends Thread {
	public void run() {
		printf("do something");
	}
}
```
<br>
注意：重写(override)run()方法在该线程的start()方法被调用后，JVM会自动调用run方法来执行任务；但是重载（overload）run()方法，该方法和普通的成员方法一样，并不会因调用该线程的start()方法而被JVM自动运行
####实现java.lang.runnable接口
```java runnable http://www.shaojie.name
	public class MyRunnable implements Runnable{
		public void run() {
			printf("do something");
		}
	}
```

###线程的启动
#### 1）如果是继承Thread类，则：
``` java startThread http://www.shaojie.name
	MyThread thread = new MyThread();
	thread.start();
```
#### 2)如果实现Runnable接口，则：
``` java startThread http://www.shaojie.name
	MyRunnable runnable = new MyRunnable();
	Thread t = new Thread(runnable);
	t.start();		
```

### Android提供了从其它线程中访问UI线程：
* Activity.runOnUiThread(Runnable);
* View.post(Runnable);
* View.postDelayed(Runnable);

``` java load http://www.shaojie.name
public void onClick(View v) { 

    new Thread(new Runnable() { 

        public void run() { 

            final Bitmap bitmap = loadImageFromNetwork("http://example.com/image.png"); 

            mImageView.post(new Runnable() { 

                public void run() { 

                    mImageView.setImageBitmap(bitmap); 

                } 

            }); 

        } 

    }).start(); 

}
```



