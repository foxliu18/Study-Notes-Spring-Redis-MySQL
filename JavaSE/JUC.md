# 1. Lock

ReentrantLock 构造公平与非公平锁

``` java

    /**
     * Creates an instance of {@code ReentrantLock}.
     * This is equivalent to using {@code ReentrantLock(false)}.
     */
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    /**
     * Creates an instance of {@code ReentrantLock} with the
     * given fairness policy.
     *
     * @param fair {@code true} if this lock should use a fair ordering policy
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```



公平锁：可以先来后到

非公平锁：可以插队（默认）

> Synchronized 和 Lock的区别

1. Synchronized 内置的java关键字，Lock是一个Java类
2. Synchronized 无法判断获取锁的状态，Lock 可以判断是否获取到了锁
3. Synchronized 会自动释放锁， Lock 需要手动释放锁，如果不释放会死锁
4. Synchronized 线程1（获得锁，阻塞）、线程2（等待，一直等待）；Lock锁不会一直等下去；（try  Lock（））
5. Synchronized 可重入锁，不可以中断的，非公平；Lock 可重入锁，可以中断锁，默认非公平（可以自己设置）
6. Synchronized 适合少量代码同步问题，Lock 适合锁大量同步代码

> 什么是锁, 锁的到底是谁?



# 2. 生产者和消费者问题

面试: 单例模式,排序算法, 生产者消费者, 死锁

> 生产者和消费者问题 Synchrionized

```java
public class A {
    public static void main(String[] args) {
        Data data = new Data();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
               try{
                   data.increment();
               }catch (InterruptedException e){
                   e.printStackTrace();
               }
            }
        }, "A").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try{
                    data.decrement();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        }, "B").start();
    }
}

class Data{
    private int number = 0;

    // +1
    public synchronized void increment() throws InterruptedException {
        if (number != 0){
            // 等待
            this.wait();
        }
        number++;
        System.out.println(Thread.currentThread().getName()+"=>"+number);
        // 通知其他线程， 我+1完毕了
        this.notifyAll();
    }
    // -1

    public synchronized void decrement() throws InterruptedException {
        if (number == 0){
            // 等待
            this.wait();
        }
        number--;
        System.out.println(Thread.currentThread().getName()+"=>"+number);
        // 通知其他线程， 我-1完毕了
        this.notifyAll();
    }
}
```

> 问题,如果不是一个生产一个消费者, 会出虚假唤醒的问题, 需要把if换成while

``` java
public class A {
    public static void main(String[] args) {
        Data data = new Data();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
               try{
                   data.increment();
               }catch (InterruptedException e){
                   e.printStackTrace();
               }
            }
        }, "A").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try{
                    data.increment();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        }, "C").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try{
                    data.increment();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        }, "D").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try{
                    data.decrement();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        }, "B").start();
    }
}

class Data{
    private int number = 0;

    // +1
    public synchronized void increment() throws InterruptedException {
        while (number != 0){
            // 等待
            this.wait();
        }
        number++;
        System.out.println(Thread.currentThread().getName()+"=>"+number);
        // 通知其他线程， 我+1完毕了
        this.notifyAll();
    }
    // -1

    public synchronized void decrement() throws InterruptedException {
        while (number == 0){
            // 等待
            this.wait();
        }
        number--;
        System.out.println(Thread.currentThread().getName()+"=>"+number);
        // 通知其他线程， 我-1完毕了
        this.notifyAll();
    }
}
```

> JUC版生产者和消费者

通过Lock找到condition进行监视

``` java
public class B {
    public static void main(String[] args) {
        DataB data = new DataB();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try{
                    data.increment();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        }, "A").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try{
                    data.increment();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        }, "C").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try{
                    data.increment();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        }, "D").start();
        new Thread(()->{
            for (int i = 0; i < 10; i++) {
                try{
                    data.decrement();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        }, "B").start();
    }
}

class DataB
{
    private int number = 0;

    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();
    // condition.await();
    // condition.signalAll();
    // +1
    public void increment() throws InterruptedException {

        try {
            lock.lock();
            while (number != 0){
                // 等待
                condition.await();
            }
            number++;
            System.out.println(Thread.currentThread().getName()+"=>"+number);
            // 通知其他线程， 我+1完毕了
            condition.signalAll();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }

    }
    // -1

    public void decrement() throws InterruptedException {

        try {
            lock.lock();
            while (number == 0){
                // 等待
                condition.await();
            }
            number--;
            System.out.println(Thread.currentThread().getName()+"=>"+number);
            // 通知其他线程， 我-1完毕了
            condition.signalAll();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }

    }
}
```

> condition 精准的通知和环境线程

# 3. 八锁现象

new 的对象 一般同步方法 锁的就是该对象

static 方法锁的是类

# 4. 集合类不安全

> List不安全

```java
public class ListTest {
    public static void main(String[] args) {
        // 并发ArrayList 不安全
        // List<String> list = new ArrayList<>();
        /**
         * 解决方案
         * 1. 换成安全的容器 new Vector()
         * 2. List<String> list = Collections.synchronizedList(new ArrayList<>());
         * 3. List<String> list = new CopyOnWriteArrayList<>();
         */

        // copyOnWrite 写入时复制， COW策略
        // 多个线程调用list的时候， 读取的时候，固定的，写入（可能出现覆盖才做）
        // 在写入的时候避免覆盖造成数据问题
        // 读写分离
        // CopyOnWriteArrayList 没有使用synchronized，
        // 效率比使用synchronized的Vector效率高
        List<String> list = new CopyOnWriteArrayList<>();
        for (int i = 1; i <= 50; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(0, 5));
                System.out.println(list);
            }, String.valueOf(i)).start();
        }
    }
```

> Set不安全

```
/** 直接使用Set<String> set = new HashSet<String>();
 * 会报错：ConcurrentModificationException
 *  1. Set<String> set = Collections.synchronizedSet(new HashSet<>());
 *  2. Set<String> set = new CopyOnWriteArraySet<>();
  */
public class SetTest {
    public static void main(String[] args) {
        Set<String> set = new CopyOnWriteArraySet<>();
        for (int i = 1; i < 50; i++) {
            new Thread(()->{
                set.add(UUID.randomUUID().toString().substring(0, 5));
                System.out.println(set);
            }, String.valueOf(i)).start();
        }
    }
}
```

HashSet的底层是什么

```java
public HashSet() {
    map = new HashMap<>();
}
public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();
```

> Map

``` java
public class MapTest {
    public static void main(String[] args) {
        // map是new HashMap<>()这样用的吗
        // 不是，默认等价与 new HashMap<>(16, 0.75f);
        // int initialCapacity:初始容量 float loadFactor：加载银子
        // Map<String, String> map = new HashMap<>();
        Map<String, String> map = new ConcurrentHashMap<>();

        for (int i = 1; i < 30; i++) {
            new Thread(()->{
                map.put(Thread.currentThread().getName(),
                        UUID.randomUUID().toString().substring(0, 5));
                System.out.println(map);
                },String.valueOf(i)).start();
        }
    }
}
```

# 5. Callable

interface

1. 可以有返回值
2. 可以跑出异常
3. 方法不通 run(),  call()

``` java
public class CallableTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // new Thread(new Runnable()).start()
        // new Thread(new FutureTask<V>)).start()
        // new Thread(new FutureTask<V>((Callable)).start()
        MyThread myThread = new MyThread();
        FutureTask<String> futureTask = new FutureTask<String>(myThread);
        new Thread(futureTask, "A").start();
        new Thread(futureTask, "B").start(); //会被缓存
        String str = (String) futureTask.get(); //get方法可能阻塞,一般把它放到最后,或者异步通信
        System.out.println(str);
    }
}
class MyThread implements Callable<String>{
    @Override
    public String call() throws Exception {
        System.out.println("call()");
        return "1234";
    }
```

细节

1. 结果会被缓存
2. get方法会阻塞

# 6. 常用辅助类

## 1. CountDownLatch

一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。

用给定的*计数* 初始化 `CountDownLatch`。由于调用了 [`countDown()`]方法，所以在当前计数到达零之前，[`await`] 方法会一直受阻塞。之后，会释放所有等待的线程，[`await`] 的所有后续调用都将立即返回。这种现象只出现一次——计数无法被重置。如果需要重置计数，请考虑使用 [`CyclicBarrier`]。

减法计数器

``` java
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        // 总数是6，必须要执行任务的时候再使用
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 1; i < 10; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+" Go out");
                countDownLatch.countDown(); // 数量-1
            },String.valueOf(i)).start();
        }
        countDownLatch.await(); //计数器归零，向下执行

        System.out.println("close door");
    }
}
```

原理：

countDownLatch.countDown(); // 数量-1

countDownLatch.await(); //计数器归零，再向下执行

每次有线程调用countDown数量-1， 假设计数器变为0， countDownLatch.await()被唤醒继续执行

## 2. CyclicBarrier

一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier 在释放等待线程后可以重用，所以称它为*循环* 的 barrier。

加法计数器

```java
public static void main(String[] args) {

        CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{
            System.out.println("召唤神龙");
        });
        for (int i = 1; i <= 7; i++) {
            final int temp = i;
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"收集"+temp+"龙珠");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                } finally {
                }
            }).start();
        }
    }
```



## 3. Semaphore

一个计数信号量。从概念上讲，信号量维护了一个许可集。如有必要，在许可可用前会阻塞每一个 [`acquire()`]，然后再获取该许可。每个 [`release()`] 添加一个许可，从而可能释放一个正在阻塞的获取者。但是，不使用实际的许可对象，`Semaphore` 只对可用许可的号码进行计数，并采取相应的行动。

```java
public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);
        for (int i = 1; i <= 6; i++) {
            new Thread(()->{
                // acquire()
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+"抢到车位");
                    TimeUnit.SECONDS.sleep(2);
                    System.out.println(Thread.currentThread().getName()+"离开车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    // release
                    semaphore.release();
                }
            }, String.valueOf(i)).start();
        }
    }
```

作用：

多个共享资源互斥作用！并发限流，控制最大线程数

# 7. 读写锁

- `public interface **ReadWriteLock**`

`ReadWriteLock` 维护了一对相关的[`锁`](https://tool.oschina.net/uploads/apidocs/jdk-zh/java/util/concurrent/locks/Lock.html)，一个用于只读操作，另一个用于写入操作。只要没有 writer，[`读取锁`](https://tool.oschina.net/uploads/apidocs/jdk-zh/java/util/concurrent/locks/ReadWriteLock.html#readLock())可以由多个 reader 线程同时保持。[`写入锁`](https://tool.oschina.net/uploads/apidocs/jdk-zh/java/util/concurrent/locks/ReadWriteLock.html#writeLock())是独占的。

所有 `ReadWriteLock` 实现都必须保证 `writeLock` 操作的内存同步效果也要保持与相关 `readLock` 的联系。也就是说，成功获取读锁的线程会看到写入锁之前版本所做的所有更新。

```java
/**
 * 独占锁（写锁） 一次只能被一个线程占有
 * 共享锁（读锁） 多个线程可以同事占有
 * ReadWriteLock
 * 读-读 可以共存
 * 读-写 不能共存
 * 写-写 可以共存
 */
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCache myCache = new MyCache();
        for (int i = 1; i <= 5; i++) {
            final int temp = i;
            new Thread(()->{
                myCache.put(temp+"", temp+"");
            },String.valueOf(i)).start();
        }

        // 读取
        for (int i = 1; i <= 5; i++) {
            final int temp = i;
            new Thread(()->{
                myCache.get(temp+"");
            },String.valueOf(i)).start();
        }

    }

}
class MyCache{
    private volatile Map<String, Object> map = new HashMap<>();
    // 读写锁，更加细腻的控制
    private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    // 存， 写的时候只希望只有一个线程写入
    public void put(String key, Object value){
        try {
            readWriteLock.writeLock().lock();
            System.out.println(Thread.currentThread().getName()+"写入"+key);
            map.put(key, value);
            System.out.println(Thread.currentThread().getName()+"写入OK");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.writeLock().unlock();
        }

    }

    // 取， 读
    public void get(String key){

        try {
            readWriteLock.readLock().lock();
            System.out.println(Thread.currentThread().getName()+"读取"+key);
            Object o = map.get(key);
            System.out.println(Thread.currentThread().getName()+"读取OK");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.readLock().unlock();
        }

    }
}
```



# 8. 阻塞队列

队列：FIFO

写入：队列满了，就需要阻塞等待

读取：如果队列是空的，必须阻塞等待生产



接口 BlockingQueue<E>

- **类型参数：**

  `E` - 在此 collection 中保持的元素类型

- **所有超级接口：**

  [Collection](https://tool.oschina.net/uploads/apidocs/jdk-zh/java/util/Collection.html)<E>, [Iterable](https://tool.oschina.net/uploads/apidocs/jdk-zh/java/lang/Iterable.html)<E>, [Queue](https://tool.oschina.net/uploads/apidocs/jdk-zh/java/util/Queue.html)<E>

- **所有已知子接口：**

  [BlockingDeque](https://tool.oschina.net/uploads/apidocs/jdk-zh/java/util/concurrent/BlockingDeque.html)<E>

- **所有已知实现类：**

  [ArrayBlockingQueue](https://tool.oschina.net/uploads/apidocs/jdk-zh/java/util/concurrent/ArrayBlockingQueue.html), [DelayQueue](https://tool.oschina.net/uploads/apidocs/jdk-zh/java/util/concurrent/DelayQueue.html), [LinkedBlockingDeque](https://tool.oschina.net/uploads/apidocs/jdk-zh/java/util/concurrent/LinkedBlockingDeque.html), [LinkedBlockingQueue](https://tool.oschina.net/uploads/apidocs/jdk-zh/java/util/concurrent/LinkedBlockingQueue.html), [PriorityBlockingQueue](https://tool.oschina.net/uploads/apidocs/jdk-zh/java/util/concurrent/PriorityBlockingQueue.html), [SynchronousQueue](https://tool.oschina.net/uploads/apidocs/jdk-zh/java/util/concurrent/SynchronousQueue.html)

使用对列

添加、移除

四组API

| 方式     | 抛出异常  | 有返回值 | 阻塞   | 等待      |
| -------- | --------- | -------- | ------ | --------- |
| 添加     | add()     | offer()  | put()  | offer(,,) |
| 移除     | remove()  | poll()   | take() | poll(,)   |
| 检测队首 | element() | peek()   |        |           |



1. 抛出异常
2. 不会抛出异常
3. 阻塞等待
4. 超时等待

> SynchronousQueue<E>

没有容量

放入就要取出

```java
/**
 * 同步对列
 * 放入一个必须要取出才能继续放
 */
public class SynchronousQueueDemo {
    public static void main(String[] args) {
        BlockingQueue<String> blockingQueue = new SynchronousQueue<String>();
        new Thread(()->{
            try {
                System.out.println(Thread.currentThread().getName()+" put 1");
                blockingQueue.put("1");
                System.out.println(Thread.currentThread().getName()+" put 2");
                blockingQueue.put("2");
                System.out.println(Thread.currentThread().getName()+" put 3");
                blockingQueue.put("3");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "T1").start();

        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+" take "+blockingQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+blockingQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+blockingQueue.take());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "T2").start();


    }
}
```

# 9. 线程池

线程池：三大方法、七大参数、4中拒绝策略

> 池化技术

程序的运行，本质：占用系统资源，

优化资源的使用：池化技术

池化技术：事先准备好一些资源，有人要用，就来使用这些资源，用完之后还给池

**线程池的好处**：

1. 减低资源的消耗
2. 提高响应的速度
3. 方便管理

线程复用，可以控制最大并发数，管理线程

> 线程池：三大方法

``` java
// Executors 工具类：三大方法
// 使用线程池来创建线程
public class Demo1 {
    public static void main(String[] args) {
//        ExecutorService threadPool = Executors.newSingleThreadExecutor();// 单个线程
//        ExecutorService threadPool = Executors.newFixedThreadPool(5); // 创建一个固定的线程池大小
        ExecutorService threadPool = Executors.newCachedThreadPool(); // 可伸缩的

        try {
            for (int i = 0; i < 6; i++) {
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName() + " ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }
    }
}
```

> 7大参数

```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }

public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }

public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }



    public ThreadPoolExecutor(int corePoolSize, // 核心线程池大小
                              int maximumPoolSize, // 最大核心线程池大小
                              long keepAliveTime, // 超时没有人调用就会释放
                              TimeUnit unit, // 超时单位
                              BlockingQueue<Runnable>  workQueue // 阻塞队列
                              ThreadFactory threadFactory, // 线程工厂， 创建线程使用
                              RejectedExecutionHandler handler // 拒绝策略
                             ) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

> 手动创建线程池

``` java
// Executors 工具类：三大方法
// 使用线程池来创建线程
/* 拒绝策略
ThreadPoolExecutor.AbortPolicy, // 不处理抛出异常
ThreadPoolExecutor.CallerRunsPolicy, // 返回调用的线程处理
ThreadPoolExecutor.DiscardOldestPolicy, 尝试和最早的竞争 失败被忽略，不抛出异常
ThreadPoolExecutor.DiscardPolicy // 丢掉任务不抛出异常
*/
public class Demo1 {
    public static void main(String[] args) {
//        ExecutorService threadPool = Executors.newSingleThreadExecutor();// 单个线程
//        ExecutorService threadPool = Executors.newFixedThreadPool(5); // 创建一个固定的线程池大小
//        ExecutorService threadPool = Executors.newCachedThreadPool(); // 可伸缩的

        /**
         * 最大线程如何定义
         * 1、 CPU 密集型， 几核就是几个最大线程，可以保证CPU效率最高 Runtime.getRuntime().availableProcessors()
         * 2、 IO 密集型 大于判断你的程序中十分耗IO的线程
         *    程序 15 个大型任务， IO十分占资源
         */
        System.out.println(Runtime.getRuntime().availableProcessors());
        ExecutorService threadPool = new ThreadPoolExecutor(
                2,
                5,
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.CallerRunsPolicy() // 阻塞队列，最大线程池都满了，后来的任务不处理
        );

        try {
            for (int i = 1; i <= 9; i++) {
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName() + " ok");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }
    }
}
```

# 10. 四大函数式接口

lambda表达式、链式编程、函数式接口、Stream流式计算

> 函数式接口：只有一个方法的接口

## Interface Function<T,R>

- **Type Parameters:**

  `T` - the type of the input to the function

  `R` - the type of the result of the function

+ Functional Interface:

This is a functional interface and can therefore be used as the assignment target for a lambda expression or method reference.



## Interface Predicate<T>

- **Type Parameters:**

  `T` - the type of the input to the predicate

- Functional Interface:

  This is a functional interface and can therefore be used as the assignment target for a lambda expression or method reference.

## Interface Consumer<T>

- **Type Parameters:**

  `T` - the type of the input to the operation

  - All Known Subinterfaces:

    [Stream.Builder](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.Builder.html)<T>

- Functional Interface:

  This is a functional interface and can therefore be used as the assignment target for a lambda expression or method reference.

## Interface Supplier<T>

- **Type Parameters:**

  `T` - the type of results supplied by this supplier

- Functional Interface:

  This is a functional interface and can therefore be used as the assignment target for a lambda expression or method reference.

# 11. Stream流式计算

> 什么是流式计算

大数据：存储+计算

集合， MySQL本质就是存储

``` java
public class Test {
    public static void main(String[] args) {
        User u1 = new User(1, "a", 21);
        User u2 = new User(2, "b", 22);
        User u3 = new User(3, "c", 23);
        User u4 = new User(4, "d", 24);
        User u5 = new User(6, "e", 25);
				// 集合， MySQL本质就是存储	
        List<User> list = Arrays.asList(u1, u2, u3, u4, u5);
        //计算交给流
        //lambda表达式，链式编程，函数式接口，Stream计算
        list.stream()
                .filter(u -> {return u.getId()%2 == 0;})
                .filter(u -> {return u.getAge()>23;})
                .map(u -> {return  u.getName().toUpperCase();})
                .sorted((uu1, uu2) -> {return uu2.compareTo(uu1);})
                .limit(1)
                .forEach(System.out::println);
    }
}
```

![image-20210905235309029](image/jmm-problem.png