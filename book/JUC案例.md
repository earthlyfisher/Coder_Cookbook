## 普通线程处理

### 线程中断

由于在线程的运行中，通过停止的方式直接停止有些暴力，并且有可能影响正常业务的流转，所以JDK提供了一种通过中断方式来实现线程终止的方法.线程的中断相对于线程的停止来讲更加温和，他不会强制线程立即退出，而是通知线程你应该停下了，但是至于线程接到这个通知后停不停下来，那就只能由线程本身决定了.

对于线程的中断，通常有以下三个方法：

```java
interrupt() //这个方法用于向线程发送一个中断信号（也可以理解为中断线程）
  
isInterrupted() //判断线程是否被中断
  
interrupted()  //判断线程是否收到中断信号，同时清除这个信号(即如果当前收到了中断信号，下次在判断就是没有收到信号)
```

但是有个例外，`Thread.sleep()`,`Thread.await()`时接收到中断信号，会清除这个中断信号，即线程下次认为自己没有被中断.如下：

```java
public static void main(String[] args) throws InterruptedException {

		Thread thread=new Thread(){
			@Override
			public void run() {
				while(true){
					if(Thread.currentThread().isInterrupted()){
						System.out.println("线程收到中断信息，做中断工作");
						break;
					}
					
					try {
                        System.out.println("线程正常运行中");
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        System.err.println("线程在睡眠中被中断");
                    }
				}
			}
		};
		thread.start();
		Thread.sleep(1000);
		thread.interrupt();
	}
```

### 通知和唤醒

通过`object.wait()`,`object.notify()`实现线程的通知唤醒操作，下面是一个例子:

```java
public class ThreadNotify {

	public static void main(String[] args) throws InterruptedException {

		StringBuilder secret = new StringBuilder();
		Thread a = new Thread(new AThread(secret));
		Thread b = new Thread(new BThread(secret));
		a.start();
		Thread.sleep(2000);
		b.start();
	}
}

class AThread implements Runnable {

	private StringBuilder secret;

	public AThread(StringBuilder secret) {
		super();
		this.secret = secret;
	}

	@Override
	public void run() {
		synchronized (secret) {
			System.out.println("A==========I must get secret to open the door");
			try {
				secret.wait();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println("A==========I get secret:" + secret.toString() + "  and thanks secrete keeper!");
		}
	}
}

class BThread implements Runnable {

	private StringBuilder secret;

	public BThread(StringBuilder secret) {
		super();
		this.secret = secret;
	}

	@Override
	public void run() {
		synchronized (secret) {
			secret.append("天王盖地虎");
			System.out.println("B==========I will give secret to needer");
			secret.notify();
			System.out.println("B==========someone get secret");
		}
	}
}
```

通过上面，可以看出`notify`,`wait`是针对对象级的操作，所以在调用这些方法时，必须使当前线程成为对象监视器的所有者，即上面通过`synchronized`来指定当前线程是该对象监视器的所有者.

如果没有同步操作，则会报如下错误，指明当前线程不是对象监视器的所有者.错误如下：

> Exception in thread "Thread-1" java.lang.IllegalMonitorStateException
>
> 	at java.lang.Object.notify(Native Method)
> 	at cn.oracle.BThread.run(ThreadInterrupt.java:52)
> 	at java.lang.Thread.run(Thread.java:745)

##JUC

`java`并发包提供了一些更高级的工具，如`ReentrantLock`,`CountDownLatch`,`CyclicBarrier`,`Exchanger`,`Fork/Join`,
`Phaser`,`Semaphore`.
###`ReentrantLock`
`ReentrantLock`起到了锁的作用，保护代码块，同`synchronized`作用相同，但比较耗时
```java
Lock lock = new ReentrantLock();  
lock.lock();  
try {   
  // update object state  
}  
finally {  
  lock.unlock();   
}  
```
###`CountDownLatch`
`CountDownLatch`能是所有线程等待，直到其他线程结束操作，通过`await`做等待操作，直到`countDown`操作使`Latch`为0，则执行后续的操作.
```java
private static void process() {
		final CountDownLatch countDownLatch=new CountDownLatch(1);
		System.out.println("我是领导，请大家等待统一行动口号");
		for(int i=0;i<3;i++){
			final int num=i;
			new Thread(){
				@Override
				public void run() {
					System.out.println("我是线程 "+num+" ,我在等待统一行动");
					try {
						countDownLatch.await();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					System.out.println("我是线程 "+num+" ,我们开始行动了");
				}
			}.start();
		}
		//领导做准备工作
		try {
			Thread.sleep(20L);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("时间到了，各位开始行动！");
		countDownLatch.countDown();
	}
```
###`CyclicBarrier`
`CyclicBarrier`是回环栅栏，通过`await`操作等待在`barrier`，当所有线程都到达`barrier`时,会推选任一线程执行`barrierAction`.并且会执行后续的操作.`CyclicBarrier`可重复使用.
```java
private final static int GROUP_SIZE = 5;

	private final static Random random = new Random();

	public static void main(String[] args) throws InterruptedException {
		processOneGroup("分组1");
	}

	private static void processOneGroup(final String groupName) throws InterruptedException {
		final CyclicBarrrierRunnable runnable = new CyclicBarrrierRunnable("prepare", groupName);
		final CyclicBarrier CYCLIC_BARRIER = new CyclicBarrier(GROUP_SIZE, runnable);

		for (int i = 0; i < GROUP_SIZE; i++) {
			new Thread(String.valueOf(i)) {
				public void run() {
					try {
						CYCLIC_BARRIER.await();
					} catch (InterruptedException | BrokenBarrierException e1) {
						// TODO Auto-generated catch block
						e1.printStackTrace();
					}
					System.out.println("我是线程组：【" + groupName + "】,第：" + this.getName() + " 号线程,我已经准备就绪！");
					
					try {
						runnable.setType("start");
						CYCLIC_BARRIER.await();
					} catch (InterruptedException | BrokenBarrierException e1) {
						e1.printStackTrace();
					}
					
					try {
						Thread.sleep(Math.abs(random.nextInt()) % 1000);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					System.out.println("我是线程组：【" + groupName + "】,第：" + this.getName() + " 号线程,我已执行完成！");
					
					try {
						runnable.setType("end");
						CYCLIC_BARRIER.await();
					} catch (InterruptedException | BrokenBarrierException e) {
						e.printStackTrace();
					}
				}
			}.start();
		}
	}
```
###`Exchanger`
`Exchanger`主要用来作为两线程交换数据使用，当线程调用`exchange`操作使，线程进入阻塞状，当其他线程也调用时，将数据交换，执行后续操作.
```java
public static void main(String[] args) {
		final Exchanger<Integer> exchanger = new Exchanger<Integer>();
		for (int i = 0; i < 2; i++) {
			final Integer num = i;
			new Thread() {
				public void run() {
					System.out.println("我是线程：Thread_" + this.getName() + " 我的数据是：" + num);
					try {
						Integer exchangeNum = exchanger.exchange(num);
						Thread.sleep(1000);
						System.out.println(
								"我是线程：Thread_" + this.getName() + " 最初的数据为：" + num + " , 交换后的数据为：" + exchangeNum);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}.start();
		}
	}
```
###`Fork/Join`
`Fork/Join`采用分而治之的思想来管理，即首先通过分割任务，最后合并任务的方式来实现.
`Fork/Join`通过使用`ForkJoinTask`和`ForkJoinPool`来完成.
其中`ForkJoinTask`提供了以下两个子类：
. RecursiveAction：用于没有返回结果的任务。
. RecursiveTask ：用于有返回结果的任务。
让我们通过一个简单的需求来使用下`Fork／Join`框架，需求是：计算`1+2+3+4`的结果。
使用`Fork／Join`框架首先要考虑到的是如何分割任务，如果我们希望每个子任务最多执行两个数的相加，那么我们设置分割的阈值是2，由于是4个数字相加，所以`Fork／Join`框架会把这个任务`fork`成两个子任务，子任务一负责计算1+2，子任务二负责计算3+4，然后再`join`两个子任务的结果。
```java
static class CountComputer extends RecursiveTask<Integer> {

		private static final long serialVersionUID = 2245387215783142216L;

		private final static int THRESHOLD = 2;

		private int start;

		private int end;

		public CountComputer(int start, int end) {
			this.start = start;
			this.end = end;
		}

		@Override
		public Integer compute() {
             int sum=0;
             boolean canCompute=end-start<=THRESHOLD;
             if(canCompute){
            	 for(int i=start;i<=end;i++){
            		 sum+=i;
            	 }
             }else{
            	 int middle=(start+end)/2;
            	 CountComputer leftCount=new CountComputer(start, middle);
            	 CountComputer rightCount=new CountComputer(middle+1, end);
            	 
            	 //执行子任务
            	 leftCount.fork();
            	 rightCount.fork();
            	 
            	 //等待子任务结束，并得到执行结果
            	 sum=leftCount.join()+rightCount.join();
             }
			return sum;
		}
	}

	public static void main(String[] args) throws InterruptedException, ExecutionException {
		ForkJoinPool pool = new ForkJoinPool();
		CountComputer task=new CountComputer(1, 4);
		pool.submit(task);
		System.out.println(task.get());
	}
```
###`Semaphore`
`Semaphore`即信号量，通过信号量，控制对公共资源的访问.通过`acquire`等待资源的获取，`release`释放资源.
```java
private final static Semaphore MAX_SEMA_PHORE = new Semaphore(5);
	
	private final static SimpleDateFormat DEFAULT_DATE_FORMAT = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

	public static String getDateTime() {
        Date datetime = Calendar.getInstance().getTime();
        return DEFAULT_DATE_FORMAT.format(datetime);
    }
	
	public static void main(String[] args) {
		for (int i = 0; i < 20; i++) {
			final int num = i;
			final Random radom = new Random();
			new Thread() {
				public void run() {
					boolean acquired = false;
					try {
						MAX_SEMA_PHORE.acquire();
						acquired = true;
						System.out.println("我是线程：" + num + " 我获得了使用权！" + getDateTime());
						Thread.sleep(1000 + (radom.nextInt() & 5000));
						System.out.println("我是线程：" + num + " 我执行完了！" + getDateTime());
					} catch (Exception e) {
						e.printStackTrace();
					} finally {
						if (acquired) {
							MAX_SEMA_PHORE.release();
						}
					}
				}
			}.start();
		}
	}
```
