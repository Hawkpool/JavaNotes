
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


#### 