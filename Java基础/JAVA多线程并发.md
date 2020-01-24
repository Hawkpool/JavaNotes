
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

// 关闭线程池 pool.shutdown(); 
// 获取所有并发任务的运行结果
for (Future f : list) { 
// 从 Future 对象上获取任务的返回值，并输出到控制台 
	System.out.println("res：" + f.get().toString()); 
}

```

#### 基于线程池的方式
线程和数据库连接这些资源都是非常宝贵的资源。那么每次需要的时候创建，不需要的时候销 毁，是非常浪费资源的。那么我们就可以使用缓存的策略，也就是使用线程池。 