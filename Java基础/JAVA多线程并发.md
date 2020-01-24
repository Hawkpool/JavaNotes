
## JAVA并发知识库
![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/24/%7B88DD2460-CD09-420F-AFFC-AC52B195B851%7D_20200124095615-1579831000559.jpg)

### JAVA线程实现/创建方式

#### 继承 Thread类
Thread 类本质上是实现了 Runnable 接口的一个实例，代表一个线程的实例。启动线程的唯一方 法就是通过 Thread类的 start()实例方法。start()方法是一个 native 方法，它将启动一个新线程，并执行 run()方法。

```java
public class MyThread extends Thread { 
	public void run() { 
		System.out.println("MyThread.run()");
	} 
}
MyThread myThread1 = new MyThread(); myThread1.start();
```


#### 实现 Runnable 接口。
如果自己的类已经 extends 另一个类，就无法直接 extends Thread，此时，可以实现一个 Runnable 接口。
```java
public class MyThread extends OtherClass implements Runnable { 
	public void run() { 
		System.out.println("MyThread.run()");
	} 
}

//启动MyThread，需要首先实例化一个 Thread，并传入自己的MyThread实例： 
MyThread myThread = new MyThread(); 
Thread thread = new Thread(myThread); 
thread.start(); 

//事实上，当传入一个 Runnable target 参数给 Thread后，Thread 的 run()方法就会调用 target.run() 
public void run() { 
	if (target != null) { 
	target.run();
	} 
}
```
#### ExecutorService、Callable<Class>、Future 有返回值线程

有返回值的任务必须实现Callable 接口，类似的，无返回值的任务必须 Runnable 接口。

执行 Callable 任务后，可以获取一个 Future 的对象，在该对象上调用 get 就可以获取到Callable 任务返回的Object 了，再结合线程池接口 ExecutorService 就可以实现传说中有返回结果的多线程了。

```java
//创建一个线程池
ExecutorService pool = Executors.newFixedThreadPool(taskSize); 

// 创建多个有返回值的任务 
List<Future> list = new ArrayList<Future>(); 
for (int i = 0; i < taskSize; i++) { 
	Callable c = new MyCallable(i + " "); 
// 执行任务并获取 Future 对象 
	Future f = pool.submit(c); list.add(f); 
} 
// 关闭线程池 
pool.shutdown(); 
// 获取所有并发任务的运行结果
for (Future f : list) { 
// 从 Future 对象上获取任务的返回值，并输出到控制台 
	System.out.println("res：" + f.get().toString()); 
}

```

#### 基于线程池的方式
线程和数据库连接这些资源都是非常宝贵的资源。那么每次需要的时候创建，不需要的时候销毁，是非常浪费资源的。那么我们就可以使用缓存的策略，也就是使用线程池。 

```java
// 创建线程池 
ExecutorService threadPool = Executors.newFixedThreadPool(10); 
while(true) { 
threadPool.execute(new Runnable() { 
// 提交多个线程任务，并执行 
	@Override 
	public void run() { 
		System.out.println(Thread.currentThread().getName() + " is running .."); 
		try { 
			Thread.sleep(3000);
		} catch (InterruptedException e) { 
			e.printStackTrace();
		} 
		} 
	}); 
} 

```

### 4 种线程池
Java 里面线程池的顶级接口是 Executor，但是严格意义上讲 Executor 并不是一个线程池，而 只是一个执行线程的工具。真正的线程池接口是 ExecutorService。

![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/24/%7B5D7C8D29-84FB-4C5F-ADE4-892A3165B022%7D_20200124101251-1579831990105.jpg)

#### newCachedThreadPool
创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们。

对于执行 很多短期异步任务的程序而言，这些线程池通常可提高程序性能。

*调用 execute 将重用以前构造 的线程（如果线程可用）。如果现有线程没有可用的，则创建一个新线程并添加到池中。终止并 从缓存中移除那些已有 60 秒钟未被使用的线程。*

因此，长时间保持空闲的线程池不会使用任何资源。

#### newFixedThreadPool
*创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程。*

在任意点，在大 多数 nThreads 线程会处于处理任务的活动状态。

如果在所有线程处于活动状态时提交附加任务， 则在有可用线程之前，附加任务将在队列中等待。

如果在关闭前的执行期间由于失败而导致任何 线程终止，那么一个新线程将代替它执行后续的任务（如果需要）。

在某个线程被显式地关闭之前，池中的线程将一直存在。

#### newScheduledThreadPool
创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。
```java
ScheduledExecutorService scheduledThreadPool= Executors.newScheduledThreadPool(3);

scheduledThreadPool.schedule(newRunnable(){ 
	@Override 
	public void run() {
		System.out.println("延迟三秒"); 
	}
}, 3, TimeUnit.SECONDS);

scheduledThreadPool.scheduleAtFixedRate(newRunnable(){
	@Override 
	public void run() { 
		System.out.println("延迟 1 秒后每三秒执行一次");
	} 
},1,3,TimeUnit.SECONDS);
```

#### newSingleThreadExecutor
Executors.newSingleThreadExecutor()返回一个线程池（这个线程池只有一个线程）,这个线程 池可以在线程死后（或发生异常时）重新启动一个线程来替代原来的线程继续执行下去！

### 线程生命周期(状态)
当线程被创建并启动以后，它既不是一启动就进入了执行状态，也不是一直处于执行状态。 在线程的生命周期中，它要经过新建(**New**)、就绪（**Runnable**）、运行（**Running**）、阻塞 (**Blocked**)和死亡(**Dead**)5 种状态。尤其是当线程启动以后，它不可能一直"霸占"着CPU独自 运行，所以CPU需要在多条线程之间切换，于是线程状态也会多次在运行、阻塞之间切换

#### 新建状态（NEW）
当程序使用 new关键字创建了一个线程之后，该线程就处于新建状态，此时仅由 JVM为其分配 内存，并初始化其成员变量的值

#### 就绪状态（RUNNABLE）
当线程对象调用了 `start()`方法之后，该线程处于就绪状态。Java 虚拟机会为其创建方法调用栈和 程序计数器，等待调度运行。

#### 运行状态（RUNNING）：
如果处于就绪状态的线程获得了 CPU，开始执行 run()方法的线程执行体，则该线程处于运行状态。

#### 阻塞状态（BLOCKED）：
阻塞状态是指线程因为某种原因放弃了 cpu 使用权，也即让出了 cpu timeslice，暂时停止运行。 
直到线程进入可运行(runnable)状态，才有机会再次获得 cpu timeslice 转到运行(running)状 态。

阻塞的情况分三种：

>- 等待阻塞（o.wait->等待对列）： 
运行(running)的线程执行 o.wait()方法，JVM会把该线程放入等待队列(waitting queue) 中。

>- 同步阻塞(lock->锁池) 
运行(running)的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则 JVM会把该线 程放入锁池(lock pool)中。

>- 其他阻塞(sleep/join) 
运行(running)的线程执行 Thread.sleep(long ms)或 t.join()方法，或者发出了 I/O请求时， JVM会把该线程置为阻塞状态。当 sleep()状态超时、join()等待线程终止或者超时、或者 I/O 处理完毕时，线程重新转入可运行(runnable)状态。

#### 线程死亡（DEAD） 线程会以下面三种方式结束，结束后就是死亡状态。
>- 正常结束 
1. run()或 call()方法执行完成，线程正常结束。
>- 异常结束 
2. 线程抛出一个未捕获的 Exception或 Error。
>- 调用stop 
3. 直接调用该线程的 stop()方法来结束该线程—该方法通常容易导致死锁，不推荐使用。

![title](https://raw.githubusercontent.com/Hawkpool/Hawk-s/master/gitNote/2020/01/24/%7BE2AB8A48-4A60-4565-95EE-D93C4F974771%7D_20200124105056-1579834270983.jpg)

























