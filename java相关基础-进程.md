# Java并发编程
java.uitl.concurrent.ThreadPoolExecutor类是线程池中最核心的一个类，因此如果要透彻地了解Java中的线程池，必须先了解这个类.
在ThreadPoolExecutor类中提供了四个构造方法：
```java 
public class ThreadPoolExecutor extends AbstractExecutorService {
    .....
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
        BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler);
    ...
}
```
下面解释下一下构造器中各个参数的含义：
- corePoolSize：核心池的大小，这个参数跟后面讲述的线程池的实现原理有非常大的关系。在创建了线程池后，默认情况下，线程池中并没有任何线程，而是等待有任务到来才创建线程去执行任务，除非调用了prestartAllCoreThreads()或者prestartCoreThread()方法，从这2个方法的名字就可以看出，是预创建线程的意思，即在没有任务到来之前就创建corePoolSize个线程或者一个线程。默认情况下，在创建了线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中；
- maximumPoolSize：线程池最大线程数，这个参数也是一个非常重要的参数，它表示在线程池中最多能创建多少个线程；
- keepAliveTime：表示线程没有任务执行时最多保持多久时间会终止。默认情况下，只有当线程池中的线程数大于corePoolSize时，keepAliveTime才会起作用，直到线程池中的线程数不大于corePoolSize，即当线程池中的线程数大于corePoolSize时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。但是如果调用了allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0；
- unit：参数keepAliveTime的时间单位，有7种取值，在TimeUnit类中有7种静态属性：
```java 
TimeUnit.DAYS;               //天
TimeUnit.HOURS;             //小时
TimeUnit.MINUTES;           //分钟
TimeUnit.SECONDS;           //秒
TimeUnit.MILLISECONDS;      //毫秒
TimeUnit.MICROSECONDS;      //微妙
TimeUnit.NANOSECONDS;       //纳秒
```



# Java创建线程的四种方式
## 继承Thread类实现多线程
## 覆写Runnable()接口实现多线程。
## 覆写Callable接口实现多线程（JDK1.5）

```java
public class MyThread implements Callable<String> {
	private int count = 20;
 
	@Override
	public String call() throws Exception {
		for (int i = count; i > 0; i--) {
//			Thread.yield();
			System.out.println(Thread.currentThread().getName()+"当前票数：" + i);
		}
		return "sale out";
	} 
 
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		Callable<String> callable  =new MyThread();
		FutureTask <String>futureTask=new FutureTask<>(callable);
		Thread mThread=new Thread(futureTask);
		Thread mThread2=new Thread(futureTask);
		Thread mThread3=new Thread(futureTask);
//		mThread.setName("hhh");
		mThread.start();
		mThread2.start();
		mThread3.start();
		System.out.println(futureTask.get());
		
	}
}
```

## 通过线程池启动多线程：通过Executor 的工具类可以创建三种类型的普通线程池：
- 4.1:FixThreadPool(int n); 固定大小的线程池,使用于为了满足资源管理需求而需要限制当前线程数量的场合。使用于负载比较重的服务器。
 ```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
public class Test {
	public static void main(String[] args) {
		ExecutorService ex=Executors.newFixedThreadPool(5);
		
		for(int i=0;i<5;i++) {
			ex.submit(new Runnable() {
				@Override
				public void run() {
					for(int j=0;j<10;j++) {
						System.out.println(Thread.currentThread().getName()+j);
					}
					
				}
			});
		}
		ex.shutdown();
	}	
}

 ```
 - 4.2 SingleThreadPoolExecutor :单线程池,需要保证顺序执行各个任务的场景 
 ```java
 import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
 
public class Test {
	public static void main(String[] args) {
		ExecutorService ex=Executors.newSingleThreadExecutor();
		
		for(int i=0;i<5;i++) {
			ex.submit(new Runnable() {
				@Override
				public void run() {
					for(int j=0;j<10;j++) {
						System.out.println(Thread.currentThread().getName()+j);
					}
					
				}
			});
		}
		ex.shutdown();
	}	
}
 ```
 
- 4.3 SingleThreadPoolExecutor :单线程池：需要保证顺序执行各个任务的场景.
- 4.4 CashedThreadPool(); 缓存线程池 当提交任务速度高于线程池中任务处理速度时，缓存线程池会不断的创建线, 适用于提交短期的异步小程序，以及负载较轻的服务器

> runbale 和 callable 的区别 ：从Java 5开始，Java提供了Callable接口。提供了一个call方法可以作为线程执行体，但是call方法可以有返回值，可以声明抛出异常。Java5提供了Future接口来代表call方法的返回值，并为Future提供了一个FutureTask实现类，该实现类实现了Future和Runnable。
