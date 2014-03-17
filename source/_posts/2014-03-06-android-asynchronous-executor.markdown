---
layout: post
title: "Android Asynchronous Executor"
date: 2014-03-06 00:22:27 +0800
comments: true
categories: Android
---
Executor框架便是Java 5中引入的，其内部使用了线程池机制，它在java.util.cocurrent 包下，通过该框架来控制线程的启动、执行和关闭，可以简化并发编程的操作。因此，在Java 5之后，通过Executor来启动线程比使用Thread的start方法更好，除了更易管理，效率更好（用线程池实现，节约开销）外，还有关键的一点：有助于避免this逃逸问题——如果我们在构造器中启动一个线程，因为另一个任务可能会在构造器结束之前开始执行，此时可能会访问到初始化了一半的对象用Executor在构造器中。

Executor框架包括：线程池，Executor，Executors，ExecutorService，CompletionService，Future，Callable等。

Executor接口中之定义了一个方法execute（Runnable command），该方法接收一个Runable实例，它用来执行一个任务，任务即一个实现了Runnable接口的类。

``` java thread http://www.shaojie.name
 Executor executor = anExecutor;
 executor.execute(new RunnableTask1());
 executor.execute(new RunnableTask2());
```

你可以使用同步的方式，新任务没有启用新线程

``` java thread http://www.shaojie.name
class DirectExecutor implements Executor {
   public void execute(Runnable r) {
     r.run();
 }}
```
 
或者异步的方式，每个新任务启用新线程。

``` java thread http://www.shaojie.name
 class ThreadPerTaskExecutor implements Executor {
   public void execute(Runnable r) {
     new Thread(r).start();
 }}
 }}
```

ExecutorService接口继承自Executor接口，它提供了更丰富的实现多线程的方法
ExecutorService是线程池的一个服务，可以随时关闭线程池，是继承Executor的。Executors是个工厂类，专门创建各种线程池。

Android常用的线程池有一下几种，在Executors里面对应的方法：

####newFixedThreadPool

创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程。在任意点，在大多数 nThreads 线程会处于处理任务的活动状态。如果在所有线程处于活动状态时提交附加任务，则在有可用线程之前，附加任务将在队列中等待。如果在关闭前的执行期间由于失败而导致任何线程终止，那么一个新线程将代替它执行后续的任务（如果需要）。在某个线程被显式地关闭之前，池中的线程将一直存在。

-newFixedThreadPool与cacheThreadPool差不多，也是能reuse就用，但不能随时建新的线程
-其独特之处:任意时间点，最多只能有固定数目的活动线程存在，此时如果有新的线程要建立，只能放在另外的队列中等待，直到当前的线程中某个线程终止直接被移出池子

-和cacheThreadPool不同，FixedThreadPool没有IDLE机制（可能也有，但既然文档没提，肯定非常长，类似依赖上层的TCP或UDP IDLE机制之类的），所以FixedThreadPool多数针对一些很稳定很固定的正规并发线程，多用于服务器

-从方法的源代码看，cache池和fixed 池调用的是同一个底层池，只不过参数不同:
fixed池线程数固定，并且是0秒IDLE（无IDLE）
cache池线程数支持0-Integer.MAX_VALUE(显然完全没考虑主机的资源承受能力），60秒IDLE 

``` java newFixedThreadPool http://www.shaojie.name
　　ExecutorService pool = Executors.newFixedThreadPool(2);
　　//创建实现了Runnable接口对象，Thread对象当然也实现了Runnable接口
　　Thread t1 = new MyThread();
　　Thread t2 = new MyThread();
　　Thread t3 = new MyThread();
　　Thread t4 = new MyThread();　
　　Thread t5 = new MyThread();
　　//将线程放入池中进行执行
　　pool.execute(t1);
　　pool.execute(t2);
　　pool.execute(t3);
　　pool.execute(t4);
　　pool.execute(t5);
```　　
　　

#### 单任务线程池，newSingleThreadExecutor：
创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们。对于执行很多短期异步任务的程序而言，这些线程池通常可提高程序性能。调用 execute 将重用以前构造的线程（如果线程可用）。如果现有线程没有可用的，则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。因此，长时间保持空闲的线程池不会使用任何资源。注意，可以使用 ThreadPoolExecutor 构造方法创建具有类似属性但细节不同（例如超时参数）的线程池。

-缓存型池子，先查看池中有没有以前建立的线程，如果有，就reuse.如果没有，就建一个新的线程加入池中
-缓存型池子通常用于执行一些生存期很短的异步型任务

-能reuse的线程，必须是timeout IDLE内的池中线程，缺省timeout是60s,超过这个IDLE时长，线程实例将被终止及移出池。
注意，放入CachedThreadPool的线程不必担心其结束，超过TIMEOUT不活动，其会自动被终止。

#### newSingleThreadExecutor
创建一个使用单个 worker 线程的 Executor，以无界队列方式来运行该线程。（注意，如果因为在关闭前的执行期间出现失败而终止了此单个线程，那么如果需要，一个新线程将代替它执行后续的任务）。可保证顺序地执行各个任务，并且在任意给定的时间不会有多个线程是活动的。与其他等效的 newFixedThreadPool(1) 不同，可保证无需重新配置此方法所返回的执行程序即可使用其他的线程。

-单例线程，任意时间池中只能有一个线程
-用的是和cache池和fixed池相同的底层池，但线程数目是1,0秒IDLE（无IDLE）


#### newScheduledThreadPool
-调度型线程池
-这个池子里的线程可以按schedule依次delay执行，或周期执行
``` java newScheduledThreadPool http://shaojie.name
ScheduledExecutorService //执行周期性或定时任务

schedule(Callable<V> callable, long delay, TimeUnit unit) //创建并执行在给定延迟后启用的 ScheduledFuture。

schedule(Runnable command, long delay, TimeUnit unit) //创建并执行在给定延迟后启用的一次性操作。

scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnitunit)//创建并执行一个在给定初始延迟后首次启用的定期操作，后续操作具有给定的周期；也就是将在 initialDelay 后开始执行，然后在initialDelay+period 后执行，接着在 initialDelay + 2 * period 后执行，依此类推。

scheduleWithFixedDelay(Runnable command, long initialDelay, long delay,TimeUnit unit)//创建并执行一个在给定初始延迟后首次启用的定期操作，随后，在每一次执行终止和下一次执行开始之间都存在给定的延迟。
```
 
