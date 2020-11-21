#Java基础

1. java基本数据类型（8种）
   - byte (8位，范围为-128~127)
   - short(16位)
   - int
   - long （64位）
   - float
   - double 
   - boolean
   - char
2. java序列化的方式

#Java多线程

##进程与线程

一个进程可以包含一个或多个线程，但至少会有一个线程  
多进程的优缺点：

>缺点一：创建进程比创建线程开销大，尤其是在Windows系统上；
>缺点二：进程间通信比线程间通信要慢，因为线程间通信就是读写同一个变量，速度很快。
>优点一：多进程稳定性比多线程高，因为在多进程的情况下，一个进程崩溃不会影响其他进程，而在多线程的情况下，任何一个线程崩溃会直接导致整个进程崩溃。

多线程编程的特点在于：
- 多线程经常需要读写共享数据，并且需要同步。
- 多线程模型是Java程序最基本的并发模型
##创建新线程
创建新线程并执行指定的代码，有以下两种方法：
**方法一：**从`Thread`派生一个自定义类，然后覆写`run()`方法：

```
public class Main {
    public static void main(String[] args) {
        Thread t = new MyThread();
        t.start(); // 启动新线程
    }
}

class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("start new thread!");
    }
}
```
**方法二：**创建Thread实例时，传入一个Runnable实例：

```
public class Main {
    public static void main(String[] args) {
        Thread t = new Thread(new MyRunnable());
        t.start(); // 启动新线程
    }
}

class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("start new thread!");
    }
}
```
> ps: **调用start()方法才能启动新线程**，并在新线程中执行run()方法；而直接调用run()方法，相当于调用了一个普通的Java方法，不会启动新线程

###线程的优先级
可以对线程设定优先级，高优先级线程操作系统对其调度可能更频繁，但并不能确保一定会先执行。

```
Thread.setPriority(int n) // 1~10, 默认值5
```
##线程的状态
- ```New```：新创建的线程，尚未执行；
- ```Runnable```：运行中的线程，正在执行`run()`方法的Java代码；
- ```Blocked```：运行中的线程，因为某些操作被阻塞而挂起；
- ```Waiting```：运行中的线程，因为某些操作在等待中；
- ```Timed Waiting```：运行中的线程，因为执行`sleep()`方法正在计时等待；
- ```Terminated```：线程已终止，因为`run()`方法执行完毕。

线程启动后，可以在`Runnable`、`Blocked`、`Waiting`、`Timed Waiting`这几个状态之间切换，知道最后变成`Terminated`状态，线程终止。

一个线程还可以等待另一个线程直到其运行结束

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(() -> {
            System.out.println("hello");
        });
        System.out.println("start");
        t.start();
        t.join();	//主线程将等待t表示的线程运行结束，然后继续往下执行自身线程，join(long)的重载方法也可以指定一个等待时间，超过等待时间后就不再继续等待。
        System.out.println("end");
    }
}
```

> `Thread.sleep(毫秒数)`：使当前线程休眠

**对另一个线程对象调用`join()`方法可以等待其执行结束，`join(long)`的重载方法也可以指定一个等待时间，超过等待时间后就不再继续等待**

##中断线程
中断线程就是其他线程给该线程发一个信号，该线程收到信号后结束执行`run()`方法，使得自身线程能立刻结束运行。
中断一个线程需要在其他线程中对目标线程调用```interrupt()```方法，这仅仅是向目标线程发送了中断请求，目标线程需要反复检测自身状态是否是`interrupted`状态，如果是，就立刻结束运行。
```
//将线程的中断标记设置为true，但不会停止线程。
public void interrupt() 
//测试当前线程是否处于中断状态，是返回true，并且调用后会立即将该中断标记清除为false
public static boolean interrupted() 
//测试线程是否已经中断。线程的中断状态不受该方法的影响。
public boolean isInterrupted()
```
##守护线程（Daemon Thread）
守护线程是指为其他线程服务的线程。在JVM中，所有非守护线程都执行完毕后，无论有没有守护线程，虚拟机都会自动退出，不必关心守护线程是否已结束。
创建守护线程只需在调用start()方法前，调用setDaemon(true)把该线程标记为守护线程：
```
Thread t = new MyThread();
t.setDaemon(true);
t.start();
```
***ps.守护线程不能持有任何需要关闭的资源，例如打开文件等，因为虚拟机退出时，守护线程没有任何机会来关闭文件，这会导致数据丢失。***

#线程同步
由于线程调度的不确定性，如果多个线程同时读写共享变量，会出现数据不一致的问题。所以对共享变量进行读写时，必须保证一组指令以原子方式执行：**某一个线程执行时，其他线程必须等待**。
保证一段代码的原子性就是通过加锁和解锁实现的。Java程序使用```synchronized```**关键字**对一个对象进行加锁
如何使用synchronized：
```
找出修改共享变量的线程代码块；
选择或创建一个共享实例(lockObject)作为锁；
使用synchronized(lockObject) { ... }。
```
如果一个类被设计为允许多线程正确访问，我们就说这个类就是“线程安全”的（thread-safe）
将synchronized逻辑封装起来更好
```
public void add(int n) {
    synchronized(this) { // 锁住this,执行代码时锁定当前对象。
        count += n;
    } // 解锁
}

public synchronized void add(int n) { // 锁住this
    count += n;
} // 解锁
#两种写法相同
```

##线程等待和唤醒
在synchronized内部可以调用```wait()```使线程进入等待状态；
必须在已获得锁的对象上调用```wait()```方法；
在synchronized内部可以调用```notify()```或```notifyAll()```唤醒其他等待线程；
必须在已获得锁的对象上调用```notify()```或```notifyAll()```方法；
已唤醒的线程还需要重新获得锁后才能继续执行。
```
 public synchronized void addTask(String s) {
        this.queue.add(s);
        this.notifyAll();
    }

public synchronized String getTask() throws InterruptedException {
        while (queue.isEmpty()) {
            this.wait();//要始终在while循环中wait()，并且每次被唤醒后拿到this锁就必须再次判断
        }
        return queue.remove();
    }
```
##ReentrantLock（可重入锁）
`synchronized`关键字用于加锁，但这种锁一是很重，二是获取时必须一直等待，没有额外的尝试机制
```ReentrantLock```可以替代synchronized进行同步；
```ReentrantLock```获取锁更安全；
```ReentrantLock```是Java代码实现的锁，我们就必须先获取锁，然后在finally中正确释放锁；
可以使用```tryLock()```尝试获取锁。
```
public class Counter {
    private int count;

    public void add(int n) {
        synchronized(this) {
            count += n;
        }  }  }
------------------------------------------改写-----------------------------------------
public class Counter {
    private final Lock lock = new ReentrantLock();
    private int count;

    public void add(int n) {
        lock.lock();  //获取锁
        try {
            count += n;
        } finally {
            lock.unlock();  //释放锁
        }  }  }
```
`ReentrantLock`可以使用`tryLock`尝试获取锁，下面代码在尝试获取锁的时候，最多等待1秒。如果1秒后仍未获取到锁，`tryLock()`返回`false`，程序就可以做一些额外处理，而不是无限等待下去
```
if (lock.tryLock(1, TimeUnit.SECONDS)) {...}
```
使用`ReentrantLock`后需要使用```Condition```对象来实现`synchronized`的```wait```和```notify```的功能
```
class TaskQueue {
    private final Lock lock = new ReentrantLock();
    private final Condition condition = lock.newCondition();
    private Queue<String> queue = new LinkedList<>();

    public void addTask(String s) {
        lock.lock();
        try {
            queue.add(s);
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public String getTask() {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                condition.await();
            }
            return queue.remove();
        } finally {
            lock.unlock();
        }
    }
}
```
使用`Condition`时，引用的`Condition`对象必须从`Lock`实例的`newCondition()`返回，这样才能获得一个绑定了`Lock`实例的Condition实例。Condition提供的```await()```、```signal()```、```signalAll()```原理和synchronized锁对象的```wait()```、```notify()```、```notifyAll()```是一致的，并且其行为也是一样的
##使用ReadWriteLock
使用`ReadWriteLock`可以做到：
- 只允许一个线程写入（其他线程既不能写入也不能读取）；
- 没有写入时，多个线程允许同时读（提高性能）。
我们需要创建一个ReadWriteLock实例，然后分别获取读锁和写锁：
```java
public class Counter {
    private final ReadWriteLock rwlock = new ReentrantReadWriteLock();
    private final Lock rlock = rwlock.readLock();
    private final Lock wlock = rwlock.writeLock();
    private int[] counts = new int[10];

    public void inc(int index) {
        wlock.lock(); // 加写锁
        try {
            counts[index] += 1;
        } finally {
            wlock.unlock(); // 释放写锁
        }
    }

    public int[] get() {
        rlock.lock(); // 加读锁
        try {
            return Arrays.copyOf(counts, counts.length);
        } finally {
            rlock.unlock(); // 释放读锁
        }
    }
}
```
把读写操作分别用读锁和写锁来加锁，在读取时，多个线程可以同时获得读锁，这样就大大提高了并发读的执行效率。
##使用StampedLock
**悲观锁：**使用`ReadWriteLock`，读的过程中不允许写，这是一种悲观的读锁。
**乐观锁：**使用`StampedLock`，读的过程中也允许获取写锁后写入！这样一来，我们读的数据就可能不一致，所以，需要一点额外的代码来判断读的过程中是否有写入，这种读锁是一种乐观锁。一旦有小概率的写入导致读取的数据不一致，需要能检测出来，再读一遍。
```java
public class Point {
    private final StampedLock stampedLock = new StampedLock();

    private double x;
    private double y;

    public void move(double deltaX, double deltaY) {
        long stamp = stampedLock.writeLock(); // 获取写锁
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            stampedLock.unlockWrite(stamp); // 释放写锁
        }
    }

    public double distanceFromOrigin() {
        long stamp = stampedLock.tryOptimisticRead(); // 获得一个乐观读锁
        // 注意下面两行代码不是原子操作
        // 假设x,y = (100,200)
        double currentX = x;
        // 此处已读取到x=100，但x,y可能被写线程修改为(300,400)
        double currentY = y;
        // 此处已读取到y，如果没有写入，读取是正确的(100,200)
        // 如果有写入，读取是错误的(100,400)
        if (!stampedLock.validate(stamp)) { // 检查乐观读锁后是否有其他写锁发生
            stamp = stampedLock.readLock(); // 获取一个悲观读锁
            try {
                currentX = x;
                currentY = y;
            } finally {
                stampedLock.unlockRead(stamp); // 释放悲观读锁
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }
}
```
和`ReadWriteLock`相比，写入的加锁是完全一样的。在读取时，首先通过`tryOptimisticRead()`获取一个乐观读锁，并返回版本号。接着读取完成后，通过`validate()`去验证版本号，如果在读取过程中没有写入，则版本号不变，否则再通过获取悲观读锁再读取一遍。
`StampedLock`是不可重入锁，不能在一个线程中反复获取同一个锁。
##使用Concurrent集合
`java.util.concurrent`包提供的集合接口是线程安全的。
使用`java.util.concurrent`包提供的线程安全的并发集合可以大大简化多线程编程，并且使得多线程同时读写并发集合是安全的；

| interface |     non-thread-safe     |                              thread-safe |
| --------- | :---------------------: | ---------------------------------------: |
| List      |        ArrayList        |                     CopyOnWriteArrayList |
| Map       |         HashMap         |                        ConcurrentHashMap |
| Set       |    HashSet / TreeSet    |                      CopyOnWriteArraySet |
| Queue     | ArrayDeque / LinkedList | ArrayBlockingQueue / LinkedBlockingQueue |
| Deque     | ArrayDeque / LinkedList |                      LinkedBlockingDeque |
使用还是和非线程安全的一样
```java
Map<String, String> map = new ConcurrentHashMap<>();
```
##使用Atomic
这是`java.util.concurrent`提供了一组原子操作的封装类。Atomic类是通过无锁（lock-free）的方式实现的线程安全（thread-safe）访问。它的主要原理是利用了CAS：Compare and Set。有`AtomicInteger`、`AtomicLong`等

##线程池
频繁创建和销毁大量线程需要消耗大量时间，所以可以考虑复用一组线程执行一些小任务，而不是一个任务对应一个新线程。这种能接收大量小任务并进行分发处理的就是线程池。
简单地说，线程池内部维护了若干个线程，没有任务的时候，这些线程都处于等待状态。如果有新任务，就分配一个空闲线程执行。如果所有线程都处于忙碌状态，新任务要么放入队列等待，要么增加一个新线程进行处理。
Java标准库提供了`ExecutorService`接口表示线程池，它的典型用法如下：
```java
// 创建固定大小的线程池:
ExecutorService executor = Executors.newFixedThreadPool(3);
// 提交任务:
executor.submit(task1);
executor.submit(task2);
executor.submit(task3);
executor.submit(task4);
executor.shutdown();
//一次性放入4个任务，由于线程池只有固定的3个线程，因此，前3个任务会同时执行
//等到有线程空闲后，才会执行后面的两个任务
```
线程池在程序结束的时候要关闭。使用`shutdown()`方法关闭线程池的时候，它会等待正在执行的任务先完成，然后再关闭。`shutdownNow()`会立刻停止正在执行的任务，`awaitTermination()`则会等待指定的时间让线程池关闭。
对于`ExecutorService`接口，Java标准库提供的几个常用实现类：

- `FixedThreadPool`：线程数固定的线程池；
- `CachedThreadPool`：线程数根据任务动态调整的线程池；
- `SingleThreadExecutor`：仅单线程执行的线程池。


###ScheduledThreadPool
 如果任务需要定期反复执行，可以使用`ScheduledThreadPool`。放入`ScheduledThreadPool`的任务可以定期反复执行。
```java
ScheduledExecutorService ses = Executors.newScheduledThreadPool(4);
// 1秒后执行一次性任务:
ses.schedule(new Task("one-time"), 1, TimeUnit.SECONDS);
// 2秒后开始执行定时任务，每3秒执行:
ses.scheduleAtFixedRate(new Task("fixed-rate"), 2, 3, TimeUnit.SECONDS);
// 2秒后开始执行定时任务，以3秒为间隔执行:
ses.scheduleWithFixedDelay(new Task("fixed-delay"), 2, 3, TimeUnit.SECONDS);
```
FixedRate是指任务总是以固定时间间隔触发，不管任务执行多长时间；
FixedDelay是指，上一次任务执行完毕后，等待固定的时间间隔，再执行下一次任务
##使用Future和Callable接口
和`Runnable`接口相比，`Callable`接口多了一个返回值，可以返回指定类型的结果，使用`call()`函数作为执行体
```java
class Task implements Callable<String> {
    public String call() throws Exception {
        return longTimeCalculation(); 
    }
}
```

一个`Future<V>`接口表示一个未来可能会返回的结果，它定义的方法有：
- `get()`：获取结果（可能会等待）
- `get(long timeout, TimeUnit unit)`：获取结果，但只等待指定的时间；
- `cancel(boolean mayInterruptIfRunning)`：取消当前任务；
- `isDone()`：判断任务是否已完成。
`ExecutorService.submit()`方法，会返回一个Future类型，一个`Future`类型的实例代表一个未来能获取结果的对象：
```java
ExecutorService executor = Executors.newFixedThreadPool(4); 
// 定义任务:
Callable<String> task = new Task();
// 提交任务并获得Future:
Future<String> future = executor.submit(task);
// 从Future获取异步执行返回的结果:
String result = future.get(); // 可能阻塞
```
##使用CompletableFuture
##使用Fork/Join
Fork/Join线程池，它可以执行一种特殊的任务：把一个大任务拆成多个小任务并行执行
具体使用参见[廖雪峰java教程](https://www.liaoxuefeng.com/wiki/1252599548343744/1306581226487842)

# Java多线程问题

1. `Thread`类中的`start()`和`run()`方法有什么区别

   - `start()`方法：1）启动一个线程；2）将线程置为`runnable`状态；3）调用线程中的`run()`方法

   - `run()`方法：只调用执行`run()`方法，并不会启动新的线程

2. `notify()`和`notifyAll()`的区别

   参见[此篇文章](https://www.jianshu.com/p/25e243850bd2?appinstall=0)

3. `cyclicbarrier`和`countdownlatch`区别

   - 线程在`countdownlatch`之后会继续执行自己的任务，而`cyclicbarrier`在所有的线程到达栅栏后才会进行后续任务，否则会阻塞
   - `countdownlatch`不可重复使用，而`cyclicbarrier`可以

4. 谈谈对`volatile`的理解

   `volatile`是java虚拟机提供的轻量级的同步机制，它保证可见性、不保证原子性、禁止指令重排

5. 问题

# JVM问题

1. 谈谈你对JVM的理解？
2. 什么是OOM？什么是StackOverFlow？有哪些方法分析？
3. JVM常用参数调优？
4. JVM中对类加载器的认识？
5. 什么是双亲委派机制？
6. 什么是沙箱安全机制？





**集合和线程相关问题**

1. java中容器有哪些？哪些是同步容器，哪些是并发容器？
2. ArrayList和LinkedList的插入和访问的时间复杂度？
3. HashMap和TreeMap有什么区别？底层数据结构是什么？
4. HashMap如果一直Push元素会怎样？HashCode全都相同如何
5. ThreadLocal底层如何实现？
6. volitile的工作原理？
7. HashMap什么情况下会扩容？
8. HashMap push方法的执行过程？
9. HashMap检测到hash冲突后将元素插入在链表的末尾还是开头
10. 线程池的工作原理，几个重要参数？
11. 阻塞队列的作用是什么？
12. AtomicInteger为什么要用CAS而不是synchronized
13. 使用无界阻塞队列会出什么问题



**Spring相关**

Spring MVC处理请求的全过程

Spring baen装载的过程

Spring的传播特性

**JVM相关**

新生代分为几个区？使用什么算法进行垃圾回收？为什么使用这个算法？

JVM老年代和新生代的比例

YGC和FGC发生的具体场景



**其他**

java反射原理和注解原理

https和http的区别？

Linux怎么查看系统负载情况？



# MySQL

**有几种连接方式？有什么不同？**

1. 笛卡尔积
2. 左连接
3. 右连接
4. 内连接
5. 左表独有
6. 右表独有
7. 全连接
8. 交集去并集

**事务的特性？**

事务是恢复和并发控制的基本单位，有4种特性：

1. 原子性 （atomicity）:强调事务的不可分割.
2. 一致性 （consistency）:事务的执行的前后数据的完整性保持一致.
3. 隔离性 （isolation）:一个事务执行的过程中,不应该受到其他事务的干扰
4. 持久性（durability） :事务一旦结束,数据就持久到数据库

**delete、truncate、drop的区别**

DELETE语句执行删除的过程是每次从表中删除一行，并且同时将该行的删除操作作为事务记录在日志中保存以便进行进行回滚操作。

 TRUNCATE TABLE 则一次性地从表中删除所有的数据并不把单独的删除操作记录记入日志保存，删除行是不能恢复的。并且在删除的过程中不会激活与表有关的删除触发器。执行速度快。

TRUNCATE 和DELETE只删除数据， DROP则删除整个表（结构和数据）



**SQL注入**

SQL注入是指在传入SQL参数的时候，传入其他语句来改变SQL原本的查询意图

如何防止？

在代码中让SQL预编译是最佳的防止SQL注入攻击的方案

在MyBatis中尽量使用`#{xx}`的传参格式，它是预编译的、安全的，避免使用`${xx}`方式，${}是未经过预编译的，仅仅是取变量的值，是非安全的。若不得不使用`${xx}`这样的参数，要手工地做好过滤工作，来防止sql注入攻击。

##一、mysql优化

你是怎样优化mysql的？

我理解的优化主要是在有索引的情况下对mysql语句进行优化，首先用explain语句分析一下当前的SQL语句，通过type字段查看访问类型，在ref、range、index、all中尽量向更高级别调整，避免全文查找的情况。

然后是建立索引、使用索引的问题，在建立索引的时候，尽量遵循一些原则，比如小表驱动大表，左连接给左边加，右连接给右边加，在使用索引的时候还要注意一下避免索引失效，比如使用最佳左前缀，尽量不适用`!=`、`or`这些运算符，调整完之后再使用explain一步步优化



# 设计模式

## 设计模式六大原则

**1.单一原则**（Single Responsibility Principle）：一个类应该只有一个职责；

**2.里氏替换原则**（LSP liskov substitution principle）：子类型必须能够替换掉基类型而起到同样的作用。

**3.依赖倒置原则（dependence inversion principle）**：面向接口编程；（通过接口作为参数实现应用场景）

　　抽象就是接口或者抽象类，细节就是实现类

　　含义：

　　　　上层模块不应该依赖下层模块，两者应依赖其抽象；

　　　　抽象不应该依赖细节，细节应该依赖抽象；

通俗点就是说变量或者传参数，尽量使用抽象类，或者接口；

【接口负责定义public属性和方法，并且声明与其他对象依赖关系，抽象类负责公共构造部分的实现，实现类准确的实现业务逻辑】

**4.接口隔离（interface segregation principle）**：建立单一接口；（扩展为类也是一种接口，一切皆接口）

　　　定义：

　　　　a.客户端不应该依赖它不需要的接口；

　　　　b.类之间依赖关系应该建立在最小的接口上；

简单理解：复杂的接口，根据业务拆分成多个简单接口；（对于有些业务的拆分多看看适配器的应用）

　【接口的设计粒度越小，系统越灵活，但是灵活的同时结构复杂性提高，开发难度也会变大，维护性降低】　　　

**5.迪米特原则（law of demeter LOD）**：最少知道原则，尽量降低类与类之间的耦合；

一个对象应该对其他对象有最少的了解

**6.开闭原则（open closed principle）：**一个类应该对拓展开放，对修改关闭。



如何安全删除list中的元素

hashMap为什么不是线程安全



