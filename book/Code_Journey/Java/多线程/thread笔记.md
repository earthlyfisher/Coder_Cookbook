# 多线程

## 基本特性

### 优先级

- 线程的优先级具有继承性，即A线程启动B线程，B的优先级和A的相同
- 线程的优先级与代码的执行顺序无关，高优先级的总是会优先执行，即CPU尽量将执行资源让给优先级较高的线程

## 线程终止解决之道

### `stop`方法(不建议使用)

`stop`方法可以停止线程，但由于存在诸如会造成死锁，数据不一致等情况，不建议使用．

### 通过标记

通过定义`volatile`变量来标记是否需要退出线程的执行.

```java
   public volatile boolean exitFlag = false;

   while (true) {
             if (exitFlag) {
                 System.out.println("我听到指令退出了");
                 return;
             }
             a++;
             b++;
         }
```

### 通过`线程中断`

```java
　　　　try {
            while (true) {
                if (this.isInterrupted()) {
                    System.out.println("我被中断了，我要退出！");
                    throw new InterruptedException();
                }
                a++;
                b++;
            }
        } catch (InterruptedException e) {
            System.out.println("进入了中断处理处！");
            e.printStackTrace();
        }
```

中断后，尽量通过抛出`InterruptedException`的方式来处理，这样可以延续到上层，做一些各层需要做的处理.

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
*注意:中断后，尽量通过抛出`InterruptedException`的方式来处理，这样可以延续到上层，做一些各层需要做的处理*.如下

```java
　　　　try {
            while (true) {
                if (this.isInterrupted()) {
                    System.out.println("我被中断了，我要退出！");
                    throw new InterruptedException();
                }
                a++;
                b++;
            }
        } catch (InterruptedException e) {
            System.out.println("进入了中断处理处！");
            e.printStackTrace();
        }
```

## `synchronized`锁

- `synchronized`锁是可重入的，即自己可以再次获取自己的内部锁，因为不可重入锁的话，会导致死锁．
  可重入锁也支持在父子类继承的环境中．
- 出现异常，锁自动释放
- 同步不具有继承性
- 同步整个方法是有弊端的，可以通过同步代码块的方式解决，即一半同步，一半异步
- 同步是采用的对象监视器的方式，同一对象同一时刻只能有一个线程访问
- 可以通过将JVM设置为`-server`模式提高线程运行的效率，则线程一直在私有堆栈中获取值

## `ReentrantLock`可重入锁

`ReentrantLock`可重入锁功能和`synchronized`类似，但它很灵活，能灵活的处理锁的获取和释放以及
在不同条件下对对象监视的`await/signal`.其提供的方法如下：

![](../../image/ReentrantLock_method.png)

其中各方法如下：

1. `lock()`
    是阻塞式的获取锁，如果有中断操作，必须获取锁才能做中断处理
2. `lockInterruptibly()`
    能处理中断操作，改线程被中断后不需用获取锁
3. `tryLock()`
     试着获取锁，如果获取到，或者该线程本来已持有锁，则返回true，否则false

下面是利用`ReentrantLock`可重入锁实现的生产者消费者模式：

```java
     package com.earthlyfish.thread.lock;
     
     import java.util.concurrent.locks.Condition;
     import java.util.concurrent.locks.ReentrantLock;
     
     /**
      * Created by earthlyfisher on 2017/4/13.
      */
     public class CSByCondition {
     
     
         public static void main(String[] args) {
             final MiddleObject object = new MiddleObject();
     
             Thread thread = new Thread() {
                 @Override
                 public void run() {
                     for (int i = 0; i < 1000; i++) {
                         System.out.println(object.get());
                     }
                 }
             };
             thread.start();
     
             for (int i = 0; i < 1000; i++) {
                 object.put(i + "");
             }
         }
     }
     
     class MiddleObject {
         private int size = 1;
     
         private int count = 0;
     
         private int getIndex = 0;
     
         private int putIndex = 0;
     
         private final String[] items;
     
         private final ReentrantLock lock;
     
         private final Condition notEmpty;
     
         private final Condition notFull;
     
         public MiddleObject() {
             this.items = new String[size];
             lock = new ReentrantLock();
             notEmpty = lock.newCondition();
             notFull = lock.newCondition();
         }
     
         public void put(String elment) {
     
             try {
                 lock.lockInterruptibly();
                 while (count == size) {
                     notFull.await();
                 }
                 items[putIndex] = elment;
                 putIndex = ++putIndex == size ? 0 : putIndex;
                 count++;
                 notEmpty.signal();
             } catch (InterruptedException e) {
                 e.printStackTrace();
             } finally {
                 lock.unlock();
     
             }
         }
     
         public String get() {
             try {
                 lock.lockInterruptibly();
                 while (count == 0) {
                     notEmpty.await();
                 }
     
                 String element = items[getIndex];
                 items[getIndex] = null;
                 getIndex = ++getIndex == size ? 0 : getIndex;
                 count--;
     
                 notFull.signal();
     
                 return element;
             } catch (InterruptedException e) {
                 throw new RuntimeException();
             } finally {
                 lock.unlock();
             }
         }
     }
```

## 线程通信

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

>Exception in thread "Thread-1" java.lang.IllegalMonitorStateException
>
>at java.lang.Object.notify(Native Method)
>at cn.oracle.BThread.run(ThreadInterrupt.java:52)
>at java.lang.Thread.run(Thread.java:745)

### 等待通知机制

1. 通过wait/notity(notifyAll)的形式来实现
2. wait/notity(notifyAll)调用时，当前线程必须要获得该对象的对象级别锁，即只能在同步方法或方法块中调用，否则会抛出`IllegalMonitorStateException`
3. wait方法将当前线程置入预执行队列，即阻塞状态，并且在wait所在的代码行处停止执行，知道被唤醒或者被中断
4. 执行wait后，当前线程释放锁
5. 如果有多个wait线程，则由线程规划器选出一个线程被notity，并使它等待获取该对象的对象锁
6. notity调用后，知道将当前同步代码块执行完，被唤醒的wait线程才能获取当前的对象锁
7. notifyAll的作用是唤醒所有线程，至于那一个得到锁，则有优先级和分配策略所决定
8. wait(timeout)可以实现等待超时，自动唤醒
9. 可以通过pipedOutputStream／PipedInputStream实现线程的通信