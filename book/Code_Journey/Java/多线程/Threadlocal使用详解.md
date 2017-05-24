#何为ThreadLocal
ThreadLocal是线程本地变量。其为每一个线程创建了一个本地副本，每个线程可以访问自己的本地变量。
#使用实例
```java
public class ExcutorTest {
    ThreadLocal<Long> longLocal = new ThreadLocal<Long>(){
        @Override
        protected Long initialValue() {
            return Thread.currentThread().getId();
        }
    };
    ThreadLocal<String> stringLocal = new ThreadLocal<String>(){
        protected String initialValue() {
            return Thread.currentThread().getName();
        };
    };
 
     
    public void set() {
        longLocal.set(Thread.currentThread().getId());
        stringLocal.set(Thread.currentThread().getName());
    }
     
    public long getLong() {
        return longLocal.get();
    }
     
    public String getString() {
        return stringLocal.get();
    }
     
    public static void main(String[] args) throws InterruptedException {
        final ExcutorTest test = new ExcutorTest();
         
         
        test.set();
        System.out.println(test.getLong());
        System.out.println(test.getString());
     
         
        Thread thread1 = new Thread(){
            public void run() {
                System.out.println(test.getLong());
                System.out.println(test.getString());
            };
        };
        thread1.start();
        thread1.join();
         
        System.out.println(test.getLong());
        System.out.println(test.getString());
    }
}
```
**注意点**
1. 实际的通过ThreadLocal创建的副本是存储在每个线程自己的threadLocals中的
2. 为何threadLocals的类型ThreadLocalMap的键值为ThreadLocal对象，因为每个线程中可有多个threadLocal变量，就像上面代码中的longLocal和stringLocal
3. 在进行get之前，必须先set，否则会报空指针异常.解决方法可以通过先初始化,即重写initialValue方法。
#经典案例
经典的就是session管理：
```java
public class ExcutorTest {
    private static final ThreadLocal threadSession = new ThreadLocal();

    public static Session getSession() throws InfrastructureException {
        Session s = (Session) threadSession.get();
        try {
            if (s == null) {
                s = getSessionFactory().openSession();
                threadSession.set(s);
            }
        } catch (HibernateException ex) {
            throw new InfrastructureException(ex);
        }
        return s;
    }
}
```
#引用
1. [Java并发编程：深入剖析ThreadLocal](http://www.cnblogs.com/dolphin0520/p/3920407.html)
