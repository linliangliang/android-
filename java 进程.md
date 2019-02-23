#Java创建线程的四种方式
1、继承Thread类实现多线程
2、覆写Runnable()接口实现多线程。
3、覆写Callable接口实现多线程（JDK1.5）


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

4、通过线程池启动多线程：通过Executor 的工具类可以创建三种类型的普通线程池：
 4.1:FixThreadPool(int n); 固定大小的线程池,使用于为了满足资源管理需求而需要限制当前线程数量的场合。使用于负载比较重的服务器。

区别：从Java 5开始，Java提供了Callable接口。提供了一个call方法可以作为线程执行体，但是call方法可以有返回值，可以声明抛出异常。
    Java5提供了Future接口来代表call方法的返回值，并为Future提供了一个FutureTask实现类，该实现类实现了Future和Runnable。
