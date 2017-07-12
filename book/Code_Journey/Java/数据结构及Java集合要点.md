`java`集合类主要分为两大体系，继承自`Collection`的`List`和`Set`,还有自己作为根的`Map`.大体的结构如下.
```
Collection
├List
│├LinkedList
│├ArrayList
│└Vector
│　└Stack
└Set
 ├HashSet
 ├TreeSet
Map
├Hashtable
├HashMap
└TreeMap
└WeakHashMap
```

## Map

`HashMap`通过hashtable实现key-value的存储，按照key的hashCode存储，可以有null key,
`TreeMap`是有序的，通过key的Comparable比较器实现key-value的有序存储
`LinkedHashMap`也是有序的，也是通过key的hashCode存储，但是遍历是增加了有序属性，即与加入的顺序保持一致

### HashMap

`HashMap`是基于`hash`算法实现的，通过hash因子的作用，将元素"比较平均"的分散，以提高元素查找的命中率.具体的实现原理如下图：

![](../../../image/HashMap.png)

`HashMap`是非线程安全的，允许`null key`,当`threshold=capacity*loadfactor`时会扩容为`capacity<<1`,这里`capacity`为桶的数量.


以下是`put<k,v>()`方法：


```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
```


可以看到优先判断`null key`，然后计算`key`的`hash`值，通过`hash`值与`capacity`结合判断位于`Hash桶`的位置,然后判断是否有当前`key`的`Entry`,如果存在，替换`value`为新值，并返回旧的`value`，否则添加一个新的`Entry`,并且返回`null`，如果一个桶已有至少一个`Entry`，则会作为链表的第一个元素插入.

### HashTable

`HashTable`的原理与`HashMap`原理类似，只是比较特殊的所有操作加上了对当前对象操作的`synchronized`,以达到同步锁的功能，从而实现线程同步.


```java
public synchronized V put(K key, V value) {
    // Make sure the value is not null
    if (value == null) {
        throw new NullPointerException();
    }

    // Makes sure the key is not already in the hashtable.
    Entry tab[] = table;
    int hash = hash(key);
    int index = (hash & 0x7FFFFFFF) % tab.length;
    for (Entry<K,V> e = tab[index] ; e != null ; e = e.next) {
        if ((e.hash == hash) && e.key.equals(key)) {
            V old = e.value;
            e.value = value;
            return old;
        }
    }

    modCount++;
    if (count >= threshold) {
        // Rehash the table if the threshold is exceeded
        rehash();

        tab = table;
        hash = hash(key);
        index = (hash & 0x7FFFFFFF) % tab.length;
    }

    // Creates the new entry.
    Entry<K,V> e = tab[index];
    tab[index] = new Entry<>(hash, key, value, e);
    count++;
    return null;
}

 private int hash(Object k) {
        // hashSeed will be zero if alternative hashing is disabled.
        return hashSeed ^ k.hashCode();
    }
```


从上可知,`HashTable`不允许加入`null key`,`null value`.

还有一个就是，`HashTable`初始容量是11.

## ConcurrentHashMap

上面的`HashTable`的锁的粒度是基于整个hash表的，而当有大批量的数据时，性能会有严重影响，而`ConcurrentHashMap`通过对整个hash表分片的思路解决了上面遇到的问题.

思路：将一个`ConcurrentHashMap`把区间按照并发级别(concurrentLevel)，分成了若干个`segment`。默认情况下内部按并发级别为16来创建。对于每个`segment`的容量，默认情况也是16。当然并发级别(`concurrentLevel`)和每个段(`segment`)的初始容量都是可以通过构造函数设定的。

`ConcurrentHashMap`的结构图大致如下：

![](../../../image/ConcurrentHashMap.png)

从图可知就是把以前的`HashTable`分成了16份而已,将锁粒度降低了.

### `Segment`





### HashSet

`Hashset`本应不在`map`系列,但由于`Hashset`的实现是基于`HashMap`实现的，所以这里列出，

```java
// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();

/**
 * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has
 * default initial capacity (16) and load factor (0.75).
 */
public HashSet() {
    map = new HashMap<>();
}

public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```


从上可以看出,每次`add`一个元素，其实是将该元素当做一个`key`存入`Hashmap`中,`value`为一个"dummy value"(一个`Object`对象).所以其实是通过`Hashmap`保证了元素的唯一.

##LinkedList与Queue

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

可知`LinkedList`实现了`Deque`，而`Deque`是丰富了父接口`Queue`，
`Queue`是Collection接口的子类，`Collection`实现了`Iterable`,所以是可遍历的。

而`Map`没有实现`Iterable`，但是通过一系列方法如`entrySet()`等转换成集合类`Set`来实现遍历功能.

## BlockingQueue

上面说到了`Queue`，而作为`Queue`子类的`BlockingQueue`在并发操作时会大量使用,其作为一个阻塞队列，在类生产者消费者模式中作为中间临界资源有很重要的用处，不需用我们过度的关注队列的临界状态，也不需要像`wait/notify`模式中在实现中必须同步已让当前线程成为该临界对象的监视器，即在其内部实现中通过锁机制做了临界的处理，而作为使用者的我们就省心了，先看一下基本层次结构如下：

```
Queue
  ├BlockingQueue
    ├ArrayBlockingQueue
    ├LinkedBlockingQueue
    ├PriorityBlockingQueue
    ├...
```

上面是基本的实现层次，下面先上一段生产者消费者模式的使用：

```java
public class ProducerConsumerPattern {
    public static void main(String args[]){
     BlockingQueue sharedQueue = new LinkedBlockingQueue();
     Thread prodThread = new Thread(new Producer(sharedQueue));
     Thread consThread = new Thread(new Consumer(sharedQueue));
     prodThread.start();
     consThread.start();
    }
}

//Producer Class in java
class Producer implements Runnable {
    private final BlockingQueue sharedQueue;
    public Producer(BlockingQueue sharedQueue) {
        this.sharedQueue = sharedQueue;
    }
    @Override
    public void run() {
        for(int i=0; i<10; i++){
            try {
                System.out.println("Produced: " + i);
                sharedQueue.put(i);
            } catch (InterruptedException ex) {
                Logger.getLogger(Producer.class.getName()).log(Level.SEVERE, null, ex);
            }
        }
    }
}
 
//Consumer Class in Java
class Consumer implements Runnable{
    private final BlockingQueue sharedQueue;
    public Consumer (BlockingQueue sharedQueue) {
        this.sharedQueue = sharedQueue;
    }
    @Override
    public void run() {
        while(true){
            try {
                System.out.println("Consumed: "+ sharedQueue.take());
            } catch (InterruptedException ex) {
                Logger.getLogger(Consumer.class.getName()).log(Level.SEVERE, null, ex);
            }
        }
    }
}
```

是不是很简单，上面不仅省去了我们同步加锁的过程，还省去了判断队列临界状态，以及`wait/notify`(通知等待)的过程.当然这种简单的使用是基于jdk的封装，其源代码实现是通过`await/signal`的`Condition	`条件处理来实现的.

先说说几个基本的操作吧：

1. `take()`&`poll()`
   * `take()`是获取并移除队列head，如果没有会一直等待知道有可用的head。
   * `poll(long timeout,TimeUnit unit)`是获取并移除队列head，如果在超时时间没有获取到可用的head就抛出异常。
2. `put()`&`offer()`
   * 这两操作与上面的对应。
   * `put()`是向队列尾部添加一个元素，如果空间不够会一直等待，知道添加成功。
   * `offer()`是向队列尾部添加一个元素，如果空间不够抛出异常。
   * `offer()`还有一个超时的方法。

***源码分析后续补上*。**

上面的问题有引出了一个新问题：

Q：类生产者消费者模式的实现方式有几种？

A：我所知道的四种：

1. `wait/notify`模式
2. 基于`Condition`的`await/signal`方式
3. 当然是上面的`BlockingQueue`了
4. 基于管道流的实现:`PipedInputStream/PipedOutputStream`

## Arraylist

`Arraylist`的实现是基于数组的，初始数组大小为10,容量不足时扩展为原来的`1.5`倍.

## ConcurrentModificationException while Iterating over ArrayList
 ![](../../../image/ConcurrentModificationException-while-Iterating-over-ArrayList.png)

## 3 ways to find duplicate elements in an array Java
 ![](../../../image/3-ways-to-find-duplicate-elements-in-an-array-Java.png)


