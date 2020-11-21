#  一些概念

**内存泄漏（Memory Leak）**是指程序中己动态分配的堆内存由于某种原因程序未释放或无法释放，造成系统内存的浪费，导致程序运行速度减慢甚至系统崩溃等严重后果。

**内存溢出(Out Of Memory，简称OOM)**是指应用系统中存在无法回收的[内存](https://baike.baidu.com/item/内存/103614)或使用的[内存](https://baike.baidu.com/item/内存/103614)过多，最终使得程序运行要用到的[内存](https://baike.baidu.com/item/内存/103614)大于能提供的最大内存。

**脏读：**脏读即为事务1第二次读取时，读到了事务2未提交的数据。若事务2回滚，则事务1第二次读取时，读到了脏数据。

**幻读：**事务1第二次查询时，读到了事务2提交的数据。即一个事务(同一个read view)在前后两次查询同一范围的时候，后一次查询看到了前一次查询没有看到的行。

**不可重复读：**事务2在事务1第二次读取时，提交了数据。导致事务1前后两次读取的数据不一致。

# 数据结构

## 红黑树

红黑树，Red-Black Tree 「RBT」是一个自平衡(不是绝对的平衡)的二叉查找树(BST)，除了符合二叉查找树的特性外，树上的每个节点还都遵循下面的规则:

1. 节点是红色或黑色。

2. 根节点是黑色。

3. 每个叶子节点都是黑色的空节点（NIL节点）。

4. 每个红色节点的两个子节点都是黑色。(从每个叶子到根的所有路径上不能有两个连续的红色节点)

5. 从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。

### 红黑树的调整

红黑树的调整分为变色和旋转，以插入为例：

1. 插入一个新节点X，将其置为红色
2. 若X为根节点，标记为黑色
3. 若X的父节点P为红色，违反原则4，调整开始
   1. 若P的兄弟节点B也为红色，则将P和B都标记为黑色
   2. 将X和其祖父节点标记为同样的颜色红色
   3. 从祖父节点开始，重复2,3步骤向上排查
4. 若X的父节点P为红色，P的兄弟节点的颜色为黑色，选择调整开始
   1. 旋转方式和AVL数类似



##红黑树和AVL树的区别

1. 红黑树不追求"完全平衡"，即不像AVL那样要求节点的 `|balFact| <= 1`，它只要求部分达到平衡，但是提出了为节点增加颜色，红黑是用非严格的平衡来换取增删节点时候旋转次数的降低，**任何不平衡都会在三次旋转之内解决**，而AVL是严格平衡树，因此在增加或者删除节点的时候，根据不同情况，旋转的次数比红黑树要多。

2. 故引入RB-Tree是**功能、性能、空间开销的折中结果**。
   1. AVL更平衡，结构上更加直观，时间效能针对读取而言更高；维护稍慢，空间开销较大。
   2. 红黑树，读取略逊于AVL，维护强于AVL，空间开销与AVL类似，内容极多时略优于AVL，维护优于AVL。

# Java 基础

代码块的加载顺序大于构造方法

`==` 和`equals`

`==`：如果比较的是基本类型，比较的是值是否相等；如果比较的是引用类型，比较的是引用地址是否相等

`equals` :比较引用类型，如果没有覆写，equals方法就是`==` ，比较的是地址。覆写过要看具体方法中的比较内容

## 静态字段和静态方法

类中的静态字段并不属于某个实例，所有实例公用一个静态字段。

## 大数：BigInteger和BigDecimal

`BigInteger`类可以实现任意精度的整数运算

`BigDecimal`类可以实现任意精度的浮点数运算

**BigIneger常用方法**

```java
static BigInteger valueOf(long val); //静态方法，将普通数值转化为大数
BigInteger(String val);	//使用带字符串参数的构造器构造任意大的大数
int compareTo(BigInteger other);	//相等返回0，小于other返回负数，否则返回正数
//大数不可以使用算术运算符，而是使用指定方法add()、subtract()、multiply()、divide()、mod()
```

**BigDecimal常用方法**

```java
//add()、subtract()、multiply()、divide()
BigDecimal divide(BigDecimal other,RoundingMode mode); 	//如果商是一个无限循环小数，divide()方法会抛出异常，需要使用这个有四舍五入的方式(RoundingMode.HALF_UP)
int compareTo(BigDecimal other); //同上
static BigDecimal valueOf(long x);
static BigDecimal valueOf(long x,int scale); //返回等于x/10^scale 的大实数
    
```



## 数组

构造方法

```java
int[] a = new int[100];
int[] b = {1,2,3,4,5};
```

数组一旦创建，长度不能改变；如果需要经常扩展数组的大小，应该使用数组列表（array list）



```java
//数组的复制不能不能使用`=`，此操作会将两个变量引用同一个数组
int[] c = b;	//此时对数组c中的修改，b的内容也同样被修改
int[] d = Arrays.copyOf(b, 10);	//第二个参数是新数组的长度

/***********常用方法******************/
Arrays.sort(a);	//数组排序，使用优化的快速排序算法
```

**Array类**

该类包含用于操作数组的各种方法，常用方法

```java
static String toString(T[] a); 	//返回包含a中元素的一个字符串，形如[a,b,c,d]
static T[] copyOf(T[] original, int newLength);//复制数组，第二个参数是新数组的长度
static T[] copyOf(T[] original, int start, int end); //新数组的长度为end-start
static void sort(T[] a);//数组排序，使用优化的快速排序算法
static int binarySearch(T[] a,T t);	//二分查找法在有序数组中查找指定值t
static void fill(T[] a,T t); //将数组的所有元素都设置为t
static boolean equals(T[] a,T[] b); //如果两个数组大小相同并且下标对应的值都相当，返回true
```

## 日期类

Date

LocalDate





# String相关

## String

一个长度为0字符串与null并不相同

字符串常量，字符串长度不可变

```java
public final class String extends Object 
implements Serializable, Comparable<String>, CharSequence
```

##StringBuffer

可变字符串，执行效率低，线程安全

```java
public final class StringBuffer extends Object
implements Serializable, CharSequence
```



##StringBuilder

可变字符串。执行效率高，线程不安全。

```java
public final class StringBuilder extends Object
implements Serializable, CharSequence
```



经常改变内容的字符串最好不要用 String ，因为每次生成对象都会对系统性能产生影响，特别当内存中无引用对象多了以后， JVM 的 GC 就会开始工作，性能就会降低。

使用 StringBuffer 类时，每次都会对 StringBuffer 对象本身进行操作，而不是生成新的对象并改变对象引用。

如果要操作少量的数据，用String ；单线程操作大量数据，用StringBuilder ；多线程操作大量数据，用StringBuffer。

# Collection

集合类的基本接口是`Collection`接口，`public interface Collection<E> extends Iterable<E>`

这个接口有两个基本方法：

```java
- boolean add(E element);
- Iterator<E> iterator();	//返回一个迭代器对象，使用迭代器对象依次访问集合中的元素
```

**迭代器**的使用：

`Iterator`接口中有4个方法:

```java
public interface Iterator<E>{
    E next();
    boolean hasNext();
    void remove();
    default void forEachRemaining(Consumer<? super E> action);
}
```

扩展了`Iterator`接口的都可以使用`for each`循环

## List

`public interface List<E>extends Collection<E>`

### ArrayList

`public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, Serializable`

- 底层使用数组实现

- 适用于查询比较多的时候，但是插入和删除比较少的情况下
- 初始类型为`Object`，大小10，第一次扩容15，第二次扩容22，`ArrayList`扩容增加原值的一半，底层使用`Arrays.copyOf()`

####ArrayList线程不安全

1. 

###LinkedList

`public class LinkedList<E> extends AbstractSequentialList<E> implements List<E>, Deque<E>, Cloneable, Serializable`

- 底层使用双向链表实现 *(**在java中，所有链表都是双向链接的**)*
- 适用于查询比较少而插入和删除比较多

## Vector

```
public class Vector<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, Serializable
```

Vector实现了List接口，所以可以作为一个List来实现。`List<Object> list = new Vector()`

- Vector线程安全

## Set

`public interface Set<E> extends Collection<E>`

`Set`用于存储不重复的元素集合

### HashSet

```
public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, Serializable
```

`HashSet`底层使用`HashMap`实现，每个value都是一个默认的值

### TreeSet

`TreeSet`是非重复的有序集合，`TreeSet`的底层是使用`TreeMap`实现，故添加的元素必须正确实现`Comparable`接口，如果没有实现`Comparable`接口，那么创建`TreeSet`时必须传入一个`Comparator`对象。

遍历`SortedSet`按照元素的排序顺序遍历，也可以自定义排序算法。

## Queue

`public interface Queue<E> extends Collection<E>`

`LinkedList`实现了`Deque`，而`interface Deque<E> extends Queue<E>`，所以可以使用`LinkedList`来构造链表队列，也可以使用`ArrayDeque`来构造一个数组队列

```java
Queue<T> queue1 = new LinkedList<T>();		//链表队列没有容量限制
Queue<T> queue2 = new ArrayDeque<>(100);	//数组队列是有界的集合
```

- `int size()`：获取队列长度

- 通过`add()`/`offer()`方法将元素添加到队尾；
- 通过`remove()`/`poll()`从队首获取元素并删除；
- 通过`element()`/`peek()`从队首获取元素但不删除。

> 对于后三种操作，每个有两种方法。区别在于，如果操作失败，前者会抛出异常，而后者只返回false(add时)或null

## DeQue

```java
public interface Deque<E> extends Queue<E>
```

双端队列，队首队尾都可以添加元素和获取元素

`Deque<String> d = new LinkedList<>();`

**常用方法：**

- `addLast()/offerLast()` 添加元素到队尾
- `E removeFirst() / E pollFirst()` 获取队首元素并从队列中删除
- `E getFirst() / E peekFirst()`获取队首元素但并不从队列中删除
- `addFirst(E e) / offerFirst(E e)`添加元素到队首
- `E removeLast() / E pollLast()`获取队尾元素并从队列中删除
- `E getLast() / E peekLast()`获取队尾元素但并不从队列中删除

#Map

`public interface Map<K,V>` 

###Map的遍历

**遍历Key**

```java
for (String key : map.keySet())
```

**遍历Key和value**

```java
for (Map.Entry<String, Object> entry : map.entrySet()) {
            String key = entry.getKey();
            Object value = entry.getValue();}
```

>Map.Entry 是Map中的一个接口，他的用途是表示一个映射项（里面有Key和Value）



## HashMap

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable
```

### HashMap的底层实现

用了`HashTable`的思想，采用空间换时间的方法

`HashMap`底层是`class Node<K,V> implements Map.Entry<K,V>`类型的数组+链表/红黑树，默认构造方法是`new HashMap(16,0.75)`；默认为链表，一个节点冲突超过8个会将链表转换为红黑树

`HashMap `使用一个大数组来存储所有的`value`，每个`Key`会计算出唯一的`hashCode`，然后根据`hashCode`访问对应`value`

所以，作为`key`的对象一定要有自己的`hashCode`方法（String已经正确实现了`hashCode`方法）

如果`key1` 和 ` key2`为两个不同引用的对象，想从Map中取出相同的值，需要覆写该对象的`equals()`方法和`hashCode()`方法，因为Map内部会:

1. 使用`hashCode()`方法检查是否有相同的hashCode
2. 使用`equals()`方法检查是否是同一个对象

一般遵循的原则为：

> 1. `equals()`用到的用于比较的每一个字段，都必须在`hashCode()`中用于计算；`equals()`中没有使用到的字段，绝不可放在`hashCode()`中计算
>2. 如果`a`和`b`不相等，那么`a.equals(b)`一定为`false`，则`a.hashCode()`和`b.hashCode()`尽量不要相等
> 3. 如果需要通过覆写`equals()`让`a`和`b`相等，那么最好同时覆写一下`hashCode()`，以维护常规约定：值相同的对象必须有相同的`hashCode`

### HashMap的自动扩容

hashMap初始默认长度为16(0~15)，超出16会自动扩容至32，其长度总为2的n次方

扩容导致数组长度增大，会重新计算`hashCode`对应索引，所以最好创建`HashMap`时就指定容量



## TreeMap

```java
public class TreeMap<K,V> extends AbstractMap<K,V> implements NavigableMap<K,V>, Cloneable, Serializable
```

`TreeMap`会对`key`进行排序，所以使用`TreeMap`时，放入的Key必须实现`Comparable`接口。

如果作为Key的class没有实现`Comparable`接口，那么，必须在创建`TreeMap`时同时指定一个自定义排序算法：

```java
Map<Person, Integer> map = new TreeMap<>(new Comparator<Person>() {
            public int compare(Person p1, Person p2) {
                return p1.name.compareTo(p2.name);
            }
```



# 多线程和JUC(java.util.concurrent)

##Thread

```java
public class Thread extends Object implements Runnable
```

**线程的概念**

线程是操作系统能够调度的最小单位；一个进程中拥有至少一个线程

###线程的创建

创建一个线程主要有三种方式：

1. 新建一个类继承`Thread`类并覆写其`run()`方法
2. 新建一个类实现`Runnable`接口并覆写其`run()`方法
3. 新建一个类实现`Callable`接口并覆写其`call()`方法(**此方法只能在线程池中使用**)

```java
java.util.concurrent
java.util.concurrent.atomic
java.util.concurrent.locks
```

###线程的状态

操作系统中线程的状态有5种：`新建`、`就绪`、`运行`、`阻塞`、`结束`

- `New`：创建后尚未启动的线程处于这种状态
- `Runnable`：调用start()方法之后，线程就处于runnable状态。可能正在执行也可能在等操作系统为它分配执行时间
- `Blocked`：线程被阻塞，等待着获取到一个排他锁；
  - static void yield() 方法，使当前正在执行的线程向另一个线程交出运行权
- `Waiting`：无限期等待，这种状态的线程不会被分配处理器执行时间，并且需要等待被其他线程显示唤醒
  - 没有设置Timeout参数的Object::wait()方法
  - 没有设置Timeout参数的Thread::join()方法
  - LockSport::park()方法
- `Timed Waiting`：限期等待，这种状态的线程不会被分配处理器执行时间，一段时间后会由系统自动唤醒；
  - Thread::sleep()方法
  - 设置了Timeout参数的Object::wait()方法
  - 设置了Timeout参数的Thread::join()方法等
- `Terminated`：线程已终止，因为run()方法执行完毕。

**阻塞和等待状态的异同：**

1. 线程处于阻塞或等待状态时，是不活动的，不运行任何代码且消耗最少的资源
2. 当一个线程视图获取一个内部的对象锁，而这个锁正被其他线程占用，该线程就会阻塞。其他线程释放了这个锁并且线程调度器允许该线程持有这个锁时，它将变为非阻塞状态
3. 当线程等待另一个线程通知调度器出现一个条件时，这个线程进入等待状态。

**Thread常用构造方法：**

```java
public Thread(Runnable target, String name)
public Thread(Runnable target)
```



### 线程的中断

除了已经废弃的stop方法，没有办法可以强制线程终止，不过，使用`interrupt`方法可以用来请求终止一个线程。每个线程都有一个boolean标志用来标识中断状态，该方法将设置中断状态，每个线程都应该时不时检查自己的该标志。

`void interrupt()` 向线程中发送中断请求。线程的中断状态将被设置为true，如果当前线程被一个sleep调用阻塞，则抛出一个InterruptedException异常。

`static boolean interrupted()` 测试当前线程（即正在执行这个指令的线程）是否被中断，这个调用有一个副作用——它会将当前线程的中断状态重新置为false。

`boolean isInterrupted()` 测试线程是否被中断。这个调用不改变线程的中断状态







**Thread常用方法：**

```java
void join();  //等待指定的线程终止
void jion(long millis);  //在millis毫秒内等待线程终止
Thread.State getState();  //得到这个线程的状态
static void sleep(long millis);		//休眠指定的毫秒数
static void yield();	//使当前正在执行的线程向另一个线程交出运行权
void setDeamon(true);	//将一个线程转换为守护线程,该方法必须在线程启动之前调用
void setPriority(int Priority); //设置线程优先级，NORM_PRIORITY为默认优先级
```



##wait和notify

**wait()**

`wait()`方法必须在当前获取的锁对象上调用

调用`wait()`方法后，线程进入等待状态，`wait()`方法不会返回，直到将来某个时刻，线程从等待状态被其他线程唤醒后，`wait()`方法才会返回，然后，**继续执行下一条语句**。

必须在`synchronized`块中才能调用`wait()`方法，因为`wait()`方法调用时，会释放线程获得的锁，`wait()`方法返回后，线程又会重新试图获得锁。

**notify()**

随机唤醒一个正在等待的线程

**notifyAll()**

唤醒所有正在等待的线程，去争抢一个资源

## Lock

> `Interface Lock` 存在于`java.util.concurrent.locks`，有如下实现类：
>
> `ReentrantLock`,` ReentrantReadWriteLock.ReadLock`,` ReentrantReadWriteLock.WriteLock`

```
 Lock l = ...;
     l.lock();
     try {
         // access the resource protected by this lock
     } finally {
         l.unlock();
     }
```

`Lock`可代替`synchronized`使用，实现精确通知

**Lock和Synchronized的区别**

| Lock                     | Synchronized                 |
| ------------------------ | ---------------------------- |
| 可重入、不可中断、非公平 | 可重入、可中断、公平/非公平  |
| 需要获取锁，释放锁       | 自动获取释放锁               |
| 等待锁的线程会一直等待   | 当时获取锁，失败不会一直等待 |



###ReentrantLock可重入锁

可重入锁：(又名递归锁)线程可以进入任何一个它已经拥有着锁的代码块。

> 一个类中有同步方法A和同步方法B，使用A去调用B，A会给整个对象加锁，调用B的时候不用再重新等待获得此对象的锁，因为整个对象已经被该线程锁了，它可以进入任何一个它已经拥有着锁的代码块。

可重入锁 的最大优点就是可以避免死锁



使用`ReentrantLock`比直接使用`synchronized`更安全，可以替代`synchronized`进行线程同步

和`synchronized`不同的是，`ReentrantLock`可以尝试获取锁：

```java
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try {
        ...
    } finally {
        lock.unlock();	//unlock必须放在finally中
    }
}
```

上述代码在尝试获取锁的时候，最多等待1秒。如果1秒后仍未获取到锁，`tryLock()`返回`false`，程序就可以做一些额外处理，而不是无限等待下去。

所以，使用`ReentrantLock`比直接使用`synchronized`更安全，线程在`tryLock()`失败的时候不会导致死锁

###公平锁和非公平锁

`ReentrantLock`构造方法

```java
Lock lock = new ReentrantLock(); 	//不传参的时候默认参数为false，构造一个非公平锁
```

**公平锁：**是指多个线程按照申请锁的顺序来获取锁，并发环境中，每个线程在获取锁时会先查看此锁维护的等待队列，如果为空，或者当前线程是等待队列的第一个，就占有锁，否则就会加入到等待队列中，FIFO。

**非公平锁：**是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获得锁。线程上来直接尝试占有锁，如果失败，再采用公平锁的方式

Synchronized也是一种非公平锁

### Condition

```java
public interface Condition
```

`Lock`代替`synchronized`使用后，对应的，使用`Condition`的方法代替`wait`和`notify`

`Condition`提供的`await()`、`signal()`、`signalAll()`原理和`synchronized`锁对象的`wait()`、`notify()`、`notifyAll()`是一致的

使用`Condition`时，引用的`Condition`对象必须从`Lock`实例的`newCondition()`返回

```java
private final Lock lock = new ReentrantLock();
private final Condition condition = lock.newCondition();
condition.signalAll();
```

###ReadWriteLock读写锁

读写锁，相比于`Lock`在读写的时候都加锁，读写锁可以进行更细粒度的控制，使用`ReadWriteLock`可以做到：

- 只允许一个线程写入（其他线程既不能写入也不能读取）；
- 没有写入时，多个线程允许同时读（提高性能）。
- 读写互斥，写写互斥

```java
private final ReadWriteLock rwlock = new ReentrantReadWriteLock();	//读写锁的构造
private final Lock rlock = rwlock.readLock();	//读写锁分为读锁和写锁
private final Lock wlock = rwlock.writeLock();

rlock.lock() 	//加读锁，写业务
rlock.unlock() 	//释放写锁
wlock.lock()	//加写锁，写业务
wlock.unlock()	//释放读锁
```

### //StampedLock 乐观锁

乐观锁的意思就是乐观地估计读的过程中大概率不会有写入，因此被称为乐观锁。

和`ReadWriteLock`相比，改进之处在于：读的过程中也允许获取写锁后写入。

因此为保证数据一致性，需要一点额外的代码来判断读的过程中是否有写入。

一旦有小概率的写入导致读取的数据不一致，需要能检测出来，再读一遍就行。

```java
public class StampedLock extends Object implements Serializable
```

###自旋锁

是指当一个线程在获取锁的时候，如果锁已经被其它线程获取，那么该线程将循环等待，然后不断的判断锁是否能够被成功获取，直到获取到锁才会退出循环。 

好处：减少上下文切换的消耗

缺点：循环会消耗CPU

自旋锁比较适用于锁使用者保持锁时间比较短的情况     



###关于同步代码的几个建议

1. 最好既不使用Lock/Condition也不适用synchronized关键字，阻塞和唤醒意味着要切换线程状态，耗费资源。在许多情况下可以使用JUC包中的某种机制，它会为你处理所有的锁定。（如阻塞队列等
2. 如果synchronized能满足，尽量该关键字，减少出错的概率。
3. 仅在需要使用Lock提供的额外能力时使用它们。

​         

## Callable和FutureTask

```java
public interface Callable<V>
```

和`Runnable`接口相比，`Callable`接口多了一个返回值，可以返回指定类型的结果，使用`call()`函数作为执行体。

```java
    @Override		//Runnable接口的实现类
    public void run() { ... }

    @Override       //Callable<Object> 接口的实现类
    public Object call() throws Exception {
        return null;
    }

```

`Thread`的构造函数中需要传入实现`Runnable`接口的类，并没有一个参数为`Callable`的构造方法。此时需要`FutureTask`使`Callable`、`Runnable`、`Thread`联系起来。

```java
/**interface RunnableFuture<V> extends Runnable, Future<V>*/
public class FutureTask<V> extends Object implements RunnableFuture<V>
/**FutureTask的构造方法**/
FutureTask(Callable<V> callable);
FutureTask(Runnable runnable, V result);

//Future接口的方法
V get();	//获取结果，会阻塞到结果可用
V get(long time,TimeUnit unit);		//获取结果，会阻塞到结果可用或超时，不成功会抛出异常

```



流程如下：

```java
/**
1. Class A implements Callable<Object>   定义一个实现Callable接口及其方法的类
2. FutureTask futureTask = new FutureTask(new A a)  使用上述类构造FutureTask
3. new Thread(futureTask).start()	开启新线程，即使多个线程开启同一个futureTask，只会返回一次结果
4. future.get() 	获取Callable的返回值，放在最后以减少等待
**/
```

##BlockingQueue阻塞队列

```
public interface BlockingQueue<E> extends Queue<E>
```

**是什么？**

阻塞队列是一个在队列基础上又支持了两个附加操作的队列

- 当队列是空的，从队列中获取元素的线程将会被阻塞
- 当队列是满的，往队列中添加元素的线程将会被阻塞

使用阻塞队列我们不需要关心什么时候需要阻塞线程，什么时候需要唤醒线程。

**怎么用？**

阻塞队列**常用于生产者和消费者的场景**，生产者是向队列里添加元素的线程，消费者是从队列里取元素的线程。简而言之，阻塞队列是生产者用来存放元素、消费者获取元素的容器。

**BlockingQueue的实现**

- `ArrayBlockingQueue`由数组结构组成的有界阻塞队列
- `LinkedBlockingQueue`由链表结构组成的有界（但大小默认值为integer.MAX_VALUE）阻塞队列
- `PriorityBlockingQueue`支持优先级排序的无界阻塞队列
- `DelayQueue`使用优先级队列实现的延迟无界阻塞队列
- `SynchronousQueue`不存储元素的阻塞队列，也即单个元素的阻塞队列
- `LinkedTransferQueue`由链表组成的无界阻塞队列
- `LinkedBlockingDeque`由链表组成的双向阻塞队列

**核心方法：**

| 方法类型 | 会抛出异常 | 特殊值   | 阻塞   | 超时               |
| -------- | ---------- | -------- | ------ | ------------------ |
| 插入     | add(e)     | offer(e) | put(e) | offer(e,time,unit) |
| 移除     | remove()   | poll()   | take() | poll(time,unit)    |
| 检查     | element()  | peek()   | /      | /                  |

会抛出异常：当阻塞队列满时，再调用`add(e)`会抛出异常；当阻塞队列空时，再调用`remove()`方法会抛出异常

特殊值：`offer(e)` 插入，成功返回true，失败false；使用`poll()`，成功返回出队元素，没有元素返回null

阻塞：当队列满时，使用`put(e)`会一直阻塞线程直到有队列有空位或中断；反之 `take()`亦然

超时：超时之后会结束并返回`boolean`

##辅助类

###CountDownLatch

它是一种同步辅助工具，允许一个或多个线程等待其他线程中正在执行的一组操作完成。

```
public class CountDownLatch
```

**使用方式：**

```java
CountDownLatch countDownLatch = new CountDownLatch(int count); 	//count为线程数
/**业务线程**/
countDownLatch.countDown();	//结束一个线程减一个数字
countDownLatch.await();	//此句相当于一个栅栏，以上所有线程执行完才开始执行之后的语句
//其他业务
```

###CyclicBarrier

一种同步辅助工具，等一组线程全部完成某个操作后才会继续进行下一步

```java
public class CyclicBarrier
```

**使用方法**

构造方法：

```java
CyclicBarrier cyclicBarrier = new CyclicBarrier(int parties)
CyclicBarrier cyclicBarrier = new CyclicBarrier(int parties, Runnable barrierAction)
//parties为线程数
//Runnable参数，这个参数的意思是最后一个到达线程要做的任务
```

重要方法：

```java
public int await();
public int await(long timeout, TimeUnit unit);
//线程调用 await() 表示自己已经到达栅栏
//定义超时时间以及时间单位
```

使用场景：

可以用于多线程计算数据，最后合并计算结果的场景。

###Semaphore

计数信号量。信号量通常用于多个共享资源的互斥使用以及限制线程的数量

```java
public class Semaphore 
```

```java
Semaphore semaphore = new Semaphore(int permits)	//设置信号量的值

/**以下代码在线程中*/
try{
	semaphore.acquire()  //当一个线程调用acquire操作时，它要么成功获取一个信号量，信号量-1，要么一直等待，直到获取到信号量或超时
	//业务代码
}finally{
	semaphore.release()  //用完后释放一个信号量，信号量+1，然后唤醒等待的线程
}


```

###JUC.atomic原子类

java在`java.util.concurrent.atomic`中提供了具有原子性的包装类`AtomicXXX`及对应方法以供直接使用。

> AtomicBoolean、AtomicInteger、AtomicIntegerArray、AtomicIntegerFieldUpdater、AtomicLong ...

**原子引用类**

```java
public class AtomicReference<V> extends Object implements Serializable
```

泛型中传入自定义的类型，即可自定义原子引用类型

```java
 AtomicReference<Object> atomticRef = new  AtomicReference<>();
```

**带版本号的原子引用类**

此种原子引用类可以避免ABA问题

```java
public class AtomicStampedReference<V> extends Object
```

#### CAS(CompareAndSet)

CAS(CompareAndSet比较并交换)是一条CPU并发原语。是`AtomicXXX`原子类中提供的方法

它的功能是判断内存某个位置的值是否为预期值，如果是，则更改为新的值，**这个过程是原子的**

```java
public final boolean compareAndSet(Object expect, Object update) {        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);}
```

- expect：期望的共享内存中的值
- update：要修改的值

如果准备修改的时候发现共享内存中的值是expect，则可以修改并返回true，否则修改失败并返回false

**底层原理**

CAS并发原语体现在java语言中就是`sun.misc.Unsafe`类中的各个方法

自旋锁

UnSafe类：包含许多native方法，可以直接操作内存

缺点：1. 循环时间开销大；2. 只能保证一个共享变量的原子操作；3. 导致ABA问题

ABA问题：CAS只管开始和结果，数据可能在中间被其他线程修改过又修改回来了，这样操作也能成功，但在某些业务中会有影响。防止ABA问题可用时间戳/版本号



## ThreadPool线程池

**是什么？**

频繁创建和销毁大量线程需要消耗大量时间，所以可以考虑复用一组线程执行一些小任务，而不是一个任务对应一个新线程。这种能接收大量小任务并进行分发处理的就是线程池。
主要特性：线程复用、控制最大并发数、管理线程

**怎么用？**

简单地说，线程池内部维护了若干个线程，没有任务的时候，这些线程都处于等待状态。如果有新任务，就分配一个空闲线程执行。如果所有线程都处于忙碌状态，新任务要么放入队列等待，要么增加一个新线程进行处理。

Java标准库提供了`ExecutorService`接口表示线程池

```java
public interface Executor
public interface ExecutorService extends Executor
```

`Excutor`是一个用来执行提交的`runnable`任务的对象。这个接口提供了一种将任务提交与每个任务如何运行的机制分离的方法，包括线程使用、调度等细节。使用其子接口`ExecutorService`的实现表示线程池，有3个(**由于其默认扩容到最大值，alibaba开发手册不允许使用`Executors`创建，而是通过`ThreadPoolExecutor`创建，以规避资源耗尽的风险**)：

- `Excutors.newFixedThreadPool(int)` ：创建一个有固定线程数的线程池，执行长期任务性能好，队列默认扩容至`Integer.Max_VALUE`
- `Excutors.newSingleThreadExecutor()` ：一池一线程
- `Excutors.newCachedThreadPool()` ：执行很多短期异步任务，可扩容，默认可扩`Integer.Max_VALUE`

**示例代码：**

```java
// 创建固定大小的线程池，推荐使用 = new ThreadPoolExecutor(...)方法构造
ExecutorService threadPool = Executors.newFixedThreadPool(3);

// 提交任务:
threadPool.submit(task1);
threadPool.submit(task2);
threadPool.submit(task3);
threadPool.submit(task4);
threadPool.shutdown();
//一次性放入4个任务，由于线程池只有固定的3个线程，因此，前3个任务会同时执行
//等到有线程空闲后，才会执行后面的两个任务
```

**底层原理：**

以上三个接口的实现，底层都是在构造`ThreadPoolExecutor`对象，为了规避默认参数造成资源耗尽，往往使用

```java
ExecutorService threadPool = ThreadPoolExecutor(int corePoolSize,
                   int maximumPoolSize,Long keepAliveTime,
                   TimeUnit unit,BlockingQueue<Runnable> workQueue,
                   ThreadFactor threadFactory,RejectedExecutionHandler handler)
```

线程池的7大参数：

`int corePoolSize`：线程池中的常驻核心线程数

`int maximumPoolSize`：线程池中能够容纳同时执行的最大线程数，不可小于1

`Long keepAliveTime`：多余的空闲线程的存活时间，超时线程会被销毁直到只剩下`corePoolSize`为止

`TimeUnit unit`：`keepAliveTime`的单位

`BlockingQueue<Runnable> workQueue`：任务队列，被提交但尚未被执行的任务
`ThreadFactor threadFactory` ：生成线程池中工作线程的线程工厂，用于创建线程，一般默认即可

`RejectedExecutionHandler handler`：拒绝策略，当工作队列满了并且工作线程已到达`maximumPoolSize`时触发，系统带有4种拒绝策略：

- `AbortPolicy`：(默认)直接抛出异常，构造：`new ThreadPoolExecutor.AbortPolicy()`
- `CallerRunsPolicy`：调用者运行策略，将任务回退给分发者
- `DiscardPolicy`：会丢弃无法处理的任务，适用于允许任务丢失的场景
- `DiscardOldestPolicy`：抛弃队列中等待最久的任务，然后把当前任务加入队列

线程池的主要处理流程：

>  提交任务到线程池 -> 核心线程池已满 -> 放入队列 -> 队列已满 -> 逐步扩容到`maximumPoolSize` - >队列和最大线程数都满了-> 拒绝策略

使用连接池时所做的工作：

1. 构建线程池
2. 调用submit提交Runnable或callable对象
3. 保存好返回的Future对象，以便得到结果或者取消任务
4. 当不想再提交任务时，调用shotdown



### ForkJoinPool

##ThreadLocal

```java
public class ThreadLocal<T> extends Object
```

**是什么？**

`ThreadLocal`在`java.lang`包中，此类提供线程局部变量。

相当于一个容器，往里面放入的内容是线程私有的

**怎么用？**

构造方法：

```java
static final ThreadLocal<T> threadLocal = new ThreadLocal<>();
```

常用方法：

`T get()`：返回此线程局部变量的当前线程副本中的值。

`void set(T value)`：将此线程局部变量的当前线程副本设置为指定值。

`void remove()`：删除此线程局部变量的当前线程值。

`static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier)` 创建一个线程局部变量，其初始值通过调用指定的supplier生成

`static ThreadLocalRandom current()` 返回特定于当前线程的Random类的实例

```java
try {
    threadLocal.set(t);
    ...
} finally {
    threadLocal.remove();	//不remove会造成内存泄漏
}
```

**底层原理：**

每一个Thread维护一个ThreadLocalMap，这个Map的key是ThreadLocal实例本身，value是要存的值。

ThreadLocalMap是ThreadLocal的内部类，没有实现Map接口，用独立的方式实现了Map的功能，其内部的Entry也是独立实现的`Entry extends WeakReference<ThreadLocal>`

当Thread销毁的时候，ThreadLocalMap也会随之销毁，减少内存的使用。

**内存泄漏问题：**

如果不手动使用`threadLocal.remove()`，会导致key被GC，从而找不到value，而无法回收，进而导致内存泄漏

如果Thread随之销毁，可以避免内存泄漏。

即ThreadLocal会内存泄漏的根源是：由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应的key，就可能会导致内存泄漏。

**hash冲突的解决**

使用线性探测法

ThreadLocal和synchronized的区别

|        | synchronized                                                 | ThreadLocal                                                  |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 原理   | 同步机制采用“以时间换空间”的方式，只提供了一份变量，让不同的线排队访问 | 采用“以空间换时间“的方式，为每个线程都提供了一份变量的副本，从而实现同时访问互不干扰 |
| 侧重点 | 多个线程之间访问资源的同步                                   | 多线程中每个线程之间的数据相互隔离                           |

ThreadLocal可以让程序拥有更高的并发性

## 注意

### 虚假唤醒

```java
while(condition){
    this.wait();
}
```

多线程交互中，条件判断不能使用`if`，只能使用`while`。因为wait被唤醒之后会重wait的下一条语句执行，使用`while`唤醒后会重新判断循环条件，如果不成立再执行while代码块之后的代码块，成立的话继续wait；如果使用if不会重新判断

### 单例模式在多线程下不安全

使用`DCL(Double Check Lock)`双端检锁机制

防止多线程情况下的指令重排导致的将 分配内存 -> 实例化对象 -> 引用地址

变为 分配内存 -> 引用地址 -> 实例化对象 而产生的问题

实例代码：

```java
public class Singleton{
    private static volatile Singleton instance = null;
    public static Singleton getInstance(){
        if(instance == null){
            synchronized(Singleton.class){
                if(instance == null)
                    instance = new Singleton()
            }
        }
    }
}
```

### 其他并发场景下的注意事项

1. 高并发场景下，避免使用“等于”判断作为终止条件，应为并发没有控制好的情况下可能发生等值判断被击穿的情况，使用大于或小于的区间判断比较好
2. 



## ArrayList不安全

1. 故障现象：`java.util.ConcurrentModifaicationException`

2. 导致原因：没锁争抢一个资源

3. 解决方法：

   1. 使用`new Vector`构造List实现线程安全

   2. 使用`Collections.synchronizedList(new ArrayList)`工具类来构造线程安全的集合

   3. 使用写时复制`new CopyOnWriteArrayList()`构造List

      写时复制，读写分离的思想，读和写不在一个容器。往一个容器中添加元素的时候不是直接往当前容器添加，而是先将当前容器`Object[]`进行copy，复制出一个新的容器，往里面添加元素。写的时候保证只有一个线程写，读的时候可以多个一起读。

##Set不安全

1. 使用`Collections.synchronizedSet(Set<T> s)`工具类来构造线程安全的集合
2. 使用写时复制`new CopyOnWriteSet()`构造Set，其底层实际还是使用`CopyOnWriteArrayList`

##Map不安全

jdk1.8中HashMap中put操作的主函数， 其中有一行判断，如果没有hash碰撞则会直接插入元素，若在此处切换线程，会导致数据被覆盖。

HashMap在容量不够进行扩容时高并发可能出现死链，导致CPU飙升，开发过程可以使用其他数据结构或加锁来规避此风险。

1. 使用`Collections.synchronizedMap(Map<K,V> m)`工具类来构造线程安全的Map
2. 使用`new ConcurrentHashMap`构造线程安全的`HashMap`

#JUF(java.util.function)

java内置核心四大函数式接口

| 函数式接口              | 参数类型 | 返回类型 | 用途                                                         |
| ----------------------- | -------- | -------- | ------------------------------------------------------------ |
| Consumer<T>消费型接口   | T        | void     | 对类型为T的对象应用操作，实现方法：`void accept()`           |
| Supplier<T>供给型接口   | 无       | T        | 返回类型为T的对象，包含方法：`T get()`                       |
| Function<T,R>函数型接口 | T        | R        | 对类型为T的对象进行操作，并返回R类型的结果，具体实现方法为`R aply(T t)` |
| Predicate<T>断定式接口  | T        | boolean  | 确定类型为T的对象是否满足某约束，并返回boolean类型的值。具体实现方法在`boolean test(T t)` |

以下是函数式接口的实例，其它类似

```java
//函数型接口,有一个输入，有一个输出。apply函数为具体实现
Function<String,Integer> function = new Function<String, Integer>() {
    @Override
    public Integer apply(String s) {
        return s.length();
    }
};
//这两个函数相等的
Function<String,String> function = s -> {return s.length() };  //使用Lambda表示上述代码
```





流式计算java.util.stream

```
public interface Stream<T> extends BaseStream<T,Stream<T>>
```

**是什么？**

流是数据管道，用于操作数据源(集合、数组等)所生成的元素序列

这种风格将要处理的**元素集合**看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。一般用于`Collection`的实现类以增强集合的处理

特点：

- Stream自己不会存储元素
- Stream不会改变源对象，会返回一个持有结果的新Stream
- 延迟执行，即需要结果的时候才执行

**怎么用？**

基本流程：

数据源 => 流水线操作 => 结果

1. **构造Stream**

```
1.使用Object.Stream，其中Object为Collection实现类(List、Set、Map等)或数组
2.Stream.of(T... values)，向其中传入数组元素
3. Stream.generate(Supplier<String> sp); 基于Supplier创建的Stream会不断调用Supplier.get()方法来不断产生下一个元素
4. 通过其它可以返回Stream的API来提供构造
```

2. 常用操作方法

- `Stream<T> filter(Predicate<? super T> predicate)`操作：就是对一个`Stream`的所有元素一一进行测试，不满足条件的就被“滤掉”了，剩下的满足条件的元素就构成了一个新的`Stream`
- `Stream<R> map(Function<? super T, ? extends R> mapper)`：通过映射，把一个`Stream`中的内容处理之后再通过`Stream`返回
- 输出为List：`Stream.collect(Collectors.toList())`方法可以将Stream的每个元素收集到List中
- 输出为List：`Stream.collect(Collectors.toSet())`方法可以将Stream的每个元素收集到Set中
- 输出为Map：`Stream.collect(Collectors.toMap(Function<? super T, ? extends K> keyMapper,Function<? super T, ? extends U> valueMapper))`需要指定两个映射函数，分别把元素映射为key和value

2. 没



# 关键字

## final

**final 修饰变量：**

- 构建对象时必须显示指定初始值
- 这个值不应该再被修改

**final修饰方法：**

- 声明 final 方法的主要目的是防止该方法的内容被修改
- final 方法可以被子类继承，但是不能被子类重写

**final修饰类：**

- final 类不能被继承

*final 通常和 static一起使用来创建类常量。*

## abstract

**abstract修饰类：**

- 抽象类不能用来实例化对象，抽象类的存在就是为了被继承，从而实现多态和统一的管理
- 抽象类可以没有抽象方法
- 包含抽象方法的类一定是抽象类
- 继承抽象类的子类必须实现父类的所有抽象方法，除非该子类也是抽象类

**abstract修饰方法：**

- 抽象方法是一种没有任何实现的方法，其具体实现由子类提供

## static

被static关键字修饰的方法或者变量不需要依赖于对象来进行访问，只要类被加载了，就可以通过类名去进行访问

**static修饰变量：**

- 所有实例化对象共享同一个static变量

**static修饰方法：**

静态方法是使用公共内存空间的，就是说所有对象都可以直接引用，不需要创建对象再使用该方法。 

- 静态方法中不能访问类的非静态成员变量和非静态成员方法

**static修饰代码块：**

- 只会在类加载的时候执行一次
- 按照static块的顺序来执行每个static块

## synchronized

synchronized声明的对象同一时间只能被一个线程访问

-  修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象
-  修饰一个方法，被修饰的方法称为同步方法，锁的对象是方法所在的整个对象(this)，同一个时间段只能有一个线程访问对象中的一个同步方法
-  修改一个静态的方法，锁的对象是这个类（或者这个类的所有对象）
-  修改一个类，其作用的范围是synchronized后面括号括起来的部分，作用的对象是这个类的所有对象。

**缺点：**

- 锁很重
- 获取时必须一直等待，没有额外的尝试机制，可能会造成死锁。

synchronized可以保证原子性、可见性、有序性

是可重入锁，非公平锁

## volatile 

`volatile`是java虚拟机提供的轻量级的同步机制，它保证可见性、不保证原子性、禁止指令重排

当成员变量发生变化时，会强制线程将变化值回写到共享内存

可见性：被volatile关键字修饰的变量，如果值发生了变更，其他线程立马可见

指令重排：被编译器翻译为机器指令的时候，可能并不是按代码顺序。步骤之间不存在依赖关系时，指令重排的优化是允许的。

volatile解决多线程内存不可见问题时，对于一写多读，是可以解决变量同步问题的，但是对于多写，同样无法解决线程安全问题。

##transient 

将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会被序列化。

transient关键字只能修饰变量，而不能修饰方法和类。

# JVM

<img src="C:\Users\DX\Desktop\正在编辑\images\jvm.webp" style="zoom:70%;" />

##类加载器 ClassLoader

负责加载class文件，在class文件内开头有特定的文件标志`cafe babe`，将class文件字节码内容加载到内存中，并将这些内容转换成方法区中的运行时数据结构。ClassLoader只负责class文件的加载，至于它是否可以运行，则由Execution Engine决定。

<img src="C:\Users\DX\Desktop\正在编辑\images\classloader.png" alt="类加载过程" style="zoom:50%;" />

`Car.class`经过`ClassLoader`加载并初始化到JVM中成为`Car Class`模板，由此模板可以创建多个实例

**加载器的种类：**

虚拟机自带的加载器

- 启动类加载器(Bootstrap)，C++实现
- 扩展类载器(Extension) ，系统自带的后来扩展的类
- 应用程序类加载器(AppClassLoader)，加载当前应用的classpath所有类，即自定义的类

用户自定义加载器：

`java.lang.ClassLoader`的子类，用户可以定制类的加载方式

> AppClassLoader 继承自 Extension 继承自 Bootstrap

装完jdk就能使用基本的类型是因为Bootstrap加载器将`rt.jar`加载到了JVM中，`rt.jar`中有系统自带的类

**查看自己的加载器：**

`obj.getClass().getClassLoader()`会返回加载器名称，`Bootstrap`类加载器返回null

**双亲委派机制：**

> 双亲委派：如果一个类加载器收到了加载某个类的请求,则该类加载器并不会去加载该类,而是把这个请求委派给父类加载器,每一个层次的类加载器都是如此,因此所有的类加载请求最终都会传送到顶端的启动类加载器;只有当父类加载器在其搜索范围内无法找到所需的类,并将该结果反馈给子类加载器,子类加载器会尝试去自己加载。
>
> 这样保证了使用不同的类加载器最终得到的都是同一个Object对象

**沙箱安全机制**

## 运行时数据区

## 方法区

它存储每一个类的**结构信息**(类的模板Class)

永久代是方法区的一个实现

JVM规范将方法区描述为堆的一个逻辑部分，物理上和堆是分开的

## Java栈

栈管运行，堆管存储。

栈由栈帧组成

**栈帧(Stack Frame)**存储了方法的局部变量表、操作数栈、动态连接、和方法返回地址、额外的附加信息。

栈也叫栈内存，主管Java程序的运行，是在线程创建时创建，它的生命周期是跟随线程的生命期，是线程私有的，线程结束栈内存也就释放，对于栈来说不存在垃圾回收问题。

**栈中存储的内容：**

8种基本类型的变量+对象的引用变量+实例方法

**栈帧：**栈帧(Stack Frame)存储了方法的局部变量表、操作数栈、动态连接、和方法返回地址、额外的附加信息。

栈帧过多会抛出：`StackOverflowError`

##堆

栈管运行，堆管存储

所有的对象实例和数组都在堆上分配内存并存放在此；是GC管理的主要区域；在虚拟机启动时创建；线程共享，非线程安全；

Java8以后，永久代换为元空间

堆逻辑上分为三部分：新生带、老年代、永久代/元空间

堆物理上分为三部分：新生带、老年代

永久代是常驻，随时都可以访问的类。永久代是方法区的实现

###新生代

#### 伊甸园区

#### 幸存者0区

####幸存者1区

###老年代

###元空间

java8之后的元空间并不在虚拟机中而是使用本机物理内存

### 堆参数调优

如果出现`java.lang.OutOfMemoryError`异常，说明java虚拟机的堆内存不够，原因有两个：

- java虚拟机的堆内存设置不够，可通过`-Xms`、`-Xmx`来调整
- 代码中创建了大量大对象，并且长时间不能被回收

| 参数               | 含义                                   |
| ------------------ | -------------------------------------- |
| -Xms               | 设置初始分配大小，默认为物理内存的1/64 |
| -Xmx               | 最大分配内存，默认为物理内存的1/4      |
| -XX:+PrintGCDetail | 输出详细的GC处理日志                   |

GCDetail打印的内容格式：`[名称：GC前内存占用 -> GC后内存占用 (该区内存总大小)]`



复制算法：From和To区域交换内容是通过复制产生的，(用在新生代)

​	优点：没碎片

​	缺点：费空间

标记清除算法：算法分为标记和清除两个阶段，先标记处要回收的对象，然后统一回收这些对象。(用在老年代)

​	优点：

​	缺点：需要两次扫描；会产生碎片

标记整理算法：和标记清除算法相比，多了一步整理碎片，可腾出连续空间放大对象(用在老年代)

​	优点：啥都好

​	缺点：耗时



# java内存模型JMM 

JMM是一种规范，定义了程序中各个变量的访问方式

<img src="C:\Users\DX\Desktop\正在编辑\images\jmm.webp" style="zoom:70%;" />

关于同步的规定：

1. 线程解锁前，必须把共享变量的值刷新回主内存
2. 线程加锁前，必须读取主内存的最新值到自己的工作内存
3. 加锁解锁是同一把锁

#Java IO

IO基本流程：

1. 创建File类的对象，指明读入/写出的文件
2. 创建输入流和输出流的对象
3. 数据的读入和写出操作
4. 关闭资源

##File

File是java中文件和目录的抽象表示形式，File对象既可以表示文件，也可以表示目录。

**常用静态成员变量：**

`File.pathSeparator`：路径分隔符，Windows为`;` ，Linux为`:`

`File.separator`：文件名称分隔符，Windows为`\` ，Linux为`/`

**构造方法：**

```java
File file = new File(String path); 	//path可以是绝对路径/相对路径/存在/不存在
File file = new File(String parent,String child)  //把路径分为父路径和子路径
File file = new File(File parent,String child)  //父路径为File类型
```

常用方法：

`public String getAbsolutePath()`:返回此File的绝对路径字符串

`public String getPath()`：返回构造参数的路径

`public String getName()`：返回由此File表示的文件或目录的名称

`public long length()`：返回由此File表示的文件的长度

**判断功能：**

`public boolean exists()`：此File表示的文件或目录是否存在

`public boolean isDirector()`：此File表示的是否为目录

`public boolean isFile()`：此File表示的是否为文件

**创建删除功能：**

`public boolean createNewFile()`：当且仅当具有该名称的文件尚不存在时，创建一个新的空文件

`public boolean delete()`：删除由此File表示的文件或目录，文件夹中有内容或文件不存在会返回false

`public boolean mkdir()`：创建由此File表示的目录

`public boolean mkdirs()`：创建多级目录

**遍历目录功能：**

`public String[] list()`：返回一个String数组，表示该File中的所有子文件或目录

`public File[] listFiles()`：返回一个File数组，表示该File目录中的所有的子文件或目录

`public File[] listFiles(FileFilter filter)`返回一个抽象路径名数组，该数组表示目录中满足指定筛选器的文件和目录。

`public String[] listFiles(FilenameFilter filter)` 返回一个抽象路径名数组，该数组表示目录中满足指定筛选器的文件和目录。

###FileFilter和FilenameFilter

抽象路径名的过滤器

```java
@FunctionalInterface
public interface FileFilter
    
//测试指定的抽象路径名是否应包含在路径名列表中
boolean	accept(File pathname)
    
/*******************************************/
    
@FunctionalInterface
public interface FilenameFilter
public accept(File dir, String name)
```

----

- 按操作数据单位的不同分为**字节流**和**字符流**
- 按数据流的流向不同分为**输入流**和**输出流**
- 按流的角色不同分为**节点流**和**处理流**

| 抽象基类 | 输入流      | 输出流       |
| -------- | ----------- | ------------ |
| 字节流   | InputStream | OutPutStream |
| 字符流   | Reader      | Writer       |

##字节流

字节（Byte）一切文件皆为字节，常用于非文本文件。文本文件(.txt .java .c)，一个中文占3个字节，使用字节流处理过程中可能出现乱码

操作步骤：

1. 创建流对象
2. 读/写
3. 释放资源

###InputStream

输入字节流，将文件读取到程序

```java
public abstract class InputStream extends Object implements Closeable
```

是Java标准库提供的最基本的输入流，一个抽象类，定义了一些通用方法：

- `void close()`：关闭流并释放资源
- ``abstract int read()` ：从输入流中读取下一个字节的数据，返回读入的一个字符，读完返回-1
- `int	read(byte[] b)`：从输入流中读取一定数量的字节并将其存储到缓冲区数组b中。
- `int read(byte[] b, int off, int len)`：从输入流将最多len字节的数据读入字节数组。

常用方法

#### FileInputStream 

把硬盘上文件中的数据读取到程序内存中

```java
public class FileInputStream extends InputStream
```

构造方法：

```java
//通过打开与实际文件（文件系统中路径名命名的文件）的连接来创建FileInputStream。
FileInputStream(String name)   
//通过打开与实际文件（由文件系统中的文件对象文件命名的文件）的连接来创建FileInputStream。
FileInputStream(File file)  
```

常用方法：

```java
FileInputStream fis = new FileInputStream("a.txt")

//1.使用read()方法循环读取文件内容，先赋值是因为防最后止fis.read()返回-1时又操作一次
while(len = fis.read() != -1){ ... }  

//2. 
byte[] bytes = new byte[1024];
int len;
while((len = fis.read(bytes)) != -1){ fos.write(bytes,0,len)}

```









###OutputStream

输出字节流，从程序中向文件中中输出

```java
public abstract class OutputStream extends Object implements Closeable, Flushable
```

一个抽象类，是所有输出字节流类的超类，定义了一些共性的方法：

- `void close()`：关闭此输出流并释放与此流关联的所有系统资源。
- ``void flush()`：刷新此输出流并强制写出任何缓冲的输出字节。
- `void write(byte[] b)`：将指定字节数组中的b.length字节写入此输出流。
- `void write(byte[] b, int off, int len)`：将指定字节数组中从偏移量off开始的len字节写入此输出流。
- `abstract void write(int b)`：将指定的字节写入此输出流，只写入`int`最低8位表示字节的部分

**OutputStream的直接子类：**

`ByteArrayOutputStream`,` FileOutputStream`, `FilterOutputStream`, `ObjectOutputStream`, `OutputStream`,` PipedOutputStream`

#### FileOutputStream

**是什么？**

文件输出流是用于将数据写入文件或文件描述符的输出流

```java
public class FileOutputStream extends OutputStream
```

**怎么用？**

常用构造方法：

```java
FileOutputStream(String name)  //创建一个向指定名称的文件中写入数据的文件输出流
FileOutputStream(File file)  //创建一个向指定File对象中写入数据的文件输出流
/*若构造方法中的文件/路径不存在，则创建一个空的文件*/
    
FileOutputStream(File file, boolean append) 
FileOutputStream(String name, boolean append)
//追加写，true：继续在文件末尾追加写；false：覆盖源文件。默认为false
```

常用操作：

`void write(byte[] b)`：将指定字节数组中的b.length字节写入文件

`void write(byte[] b, int off, int len)`：将指定字节数组中从偏移量off开始的len字节写入此文件输出流。

##字符流

只能处理字符文件，不能处理字节文件

### Reader

`Reader`是一个字符流，以`char`为单位读取。`java.io.Reader`是所有字符输入流的超类。

```java
public abstract class Reader extends Object implements Readable, Closeable
```

常用方法：

- `int read()`：读取单个字符
- `int read(char[] cbuf)`：返回每次读入数组cbuf中字符的个数，如果达到文件末尾，返回-1

####FileReader

```java
public class FileReader extends InputStreamReader
```

判断字符流是否读取结束

```java
//第一种方法看是否读取到-1
while((len = fr.read(cbuf)) != -1){ ... }
//第二种方法按行读，看是否读到null
s = bf.readLine()) != null

char[] cbuf = new char[5]
while(len = fr.read(cbuf) != -1){ //操作成功读取的len个字符而不是数组里的全部}
```



### Writer

`Writer`是所有字符输出流的超类

```java
public abstract class Writer extends Object implements Appendable, Closeable, Flushable
```

主要方法：

- `void write(char[] cbuf)`
- `abstract void	write(char[] cbuf, int off, int len)` 
- `void write(int c)`
- `void write(String str)`
- `void write(String str, int off, int len)`

#### FileWriter

一个方便书写字符到文件类

```java
public class FileWriter extends OutputStreamWriter
```

构造方法：

```java
FileWriter(File file)
FileWriter(String fileName)
FileWriter(File file,boolean append)
FileWriter(String fileName,boolean append)
```

写入换行符：

> windows下的文本文件换行符：\r\n
> linux/unix下的文本文件换行符：\r
> Mac下的文本文件换行符：\n

*如果file不存在，会新建文件*





##缓冲流

处理流：套接在已有的流的基础上

缓存流是处理流(外层流)之一：可以提升文件的读写速度

原理：内部提供了一个缓冲区（默认为8192），先向缓冲区写，写满了再一次性写到文件

外层流关闭的同时，内层流也会自动进行关闭

```java
//BufferedInputStream构造方法
BufferedInputStream(InputStream in)
BufferedInputStream(InputStream in, int size)

//BufferedOutputStream构造方法
BufferedOutputStream(OutputStream out)
BufferedOutputStream(OutputStream out, int size)
    
//BufferedReader构造方法
BufferedReader(Reader in)
BufferedReader(Reader in, int sz)

//BufferedWriter构造方法
BufferedWriter(Writer out)
BufferedWriter(Writer out, int sz)
```

使用步骤：

1. 造文件File
2. 造流
   1. 造节点流
   2. 造缓冲流
3. 操作缓冲流
4. 关闭外层流

## 转换流

转换流提供了在字节流和字符流之间的转换

java API提供了两个转换流：

- InputStreamReader：将InputStream转换为Reader
- OutputStreamWriter：将Writer转换为OutputStream

## 对象流

- ObjectInputStream
- ObjectOutputStream

用于存储和读取基本数据类型数据或对象的处理流。它的强大之处就是可以把java中的对象写入到数据源中，也能把对象从数据源中还原出来。

序列化：用ObjectOutputStream类**保存**基本类型数据或对象的机制

反序列化：用ObjectInputStream类**读取**基本类型数据或对象的机制

对象序列化机制允许把内存中的java对象转换成平台无关的二进制流，从而允许把这种二进制流持久地保存在磁盘上，或通过网络将这种二进制流传到另一个网络节点，当其他应用程序获取了这种二进制流，就可以恢复成原来的java对象。序列化的好处在于可将任何实现了`Serializable`接口的对象转化为字节数据，使其在保存和传输时可被还原。

##Java序列化与反序列化

###Java序列化

Java序列化就是把java 对象转换为字节序列的过程，转换为字节后方便数据的传输。**凡是离开内存的信息都要进行序列化**。

**Serializable**

`Serializable`接口是一个标识接口，无任何实现方法。

注意：

1. 实现该接口的类需要有一个参数`serialVersionUID`，其目的是对序列化对象进行版本控制。最好显式地声名以防止版本出现问题导致反序列化失败

```java
 private static final long serialVersionUID = 1L;
```

2. 确保要序列化的对象内部属性也都可序列化

# 反射

反射是被视为动态语言的关键，反射机制允许程序在执行期间借助于Reflection API获取任何类的内部信息，并能直接操作任意对象的内部属性及方法。

Java虚拟机中，每个实例都会对应一个Class

`Class`类的实例表示正在运行的Java应用程序中的类和接口。

```java
public final class Class<T> extends Object
implements Serializable, GenericDeclaration, Type, AnnotatedElement
```

对任意的一个对象实例，只要我们获取了它的`Class`，就可以获取它的一切信息

以下类型有Class对象

外部类、内部类、接口、数组、枚举类、注解、基本数据类型、void 

## 常用方法

###获取Class的方式

1. 调用运行时类的属性`Class clazz = Person.class`
2. 通过运行时类的对象：`Class clazz = obj.getClass()`
3. 调用Class的静态方法：`Class.forName(String 类全限定名)` (最常用)
4. 使用类加载器：`Class clazz = classLoader.loadClass("String 类的全限定名")`

**创建对应的运行时类的对象**：

```java
Class<Person> clazz = Person.class;
//调用此方法，创建对应的运行时类的对象。内部调用了运行时类的空参构造方法
Person p = clazz.newInstance(); 
```

使用`newInstance()`方法必须满足：

1. 运行时类必须提供空参的构造器
2. 空参的构造器访问权限为public

###获取/操作属性

**Field**提供有关类或接口的单个字段的信息和动态访问。反射字段可以是类（静态）字段或实例字段。

- `Field[] getFields()`获取当前运行时类及其父类中声明为public的属性

- `Field[] getDeclaredFields()`获取当前运行时类中声明的所有属性（不包括父类中的）

- `Field getField(name)`：根据字段名获取某个public的属性（包括父类）
- `Field getDeclaredField(name)`：根据字段名获取当前类的某个属性（不包括父类）

**使用Feild操作属性**

若属性为private，使用`setAccessible()`确保其可访问

`void set(Object obj, Object value)`设置当前属性值

​	参数1：指明设置哪个对象的属性

​	参数2：要设置的值

###获取/操作方法

**Method**提供有关类或接口上的单个方法的信息和对该方法的访问。反射的方法可以是类方法或实例方法（包括抽象方法）

`Method[] getMethods()`：获取当前运行时类及其父类中声明为public的方法

`Method[] getDeclaredMethods()`：获取当前运行时类中声明的所有属性（不包括父类中的）

`Method getDeclaredMethod(String name, Class<?>... parameterTypes)` ：获取当前运行时类的指定方法

​	参数1：指明获取方法的名称

​	参数2：指明获取方法的形参列表

**使用Method获取其结构**

`int getModifiers()`：获取方法的权限修饰符

`Class<?> getReturnType()`：获取方法的返回值类型

`String getName()`：获取方法名

`Class<?>[] getParameterTypes()`：获取参数类型

**使用Method调用方法**

先使用`setAccessible()`确保其可访问

`Object invoke(Object obj, Object... args)` 调用指定方法

​	参数1：方法的调用者

​	参数2：该传入的参数

###获取/使用运行时构造器

**Constructor**提供有关类的单个构造函数的信息和对该类的单个构造函数的访问。

`Constructor<?>[] getConstructors()`：获取当前运行时类中声明为public的构造器

`Constructor<T>	getDeclaredConstructor(Class<?>... parameterTypes)`：获取指定的构造器



**Type**是Java编程语言中所有类型的通用超接口。



## 动态代理

代理模式原理：

使用一个代理将对象包装起来，然后用该代理对象取代原始对象。任何对原始对象的调用都要通过代理。代理对象决定是否以及何时将方法调用转到原始对象上。

**动态代理**是指客户通过代理类来调用其它对象的方法，并且是在程序运行时根据动态创建目标类的代理对象。

不编写实现类，直接在运行期创建某个`interface`的实例

Proxy类提供静态方法来创建动态代理类和实例。

```java
public class Proxy extends Object implements Serializable
```



返回指定接口的代理类实例，该接口将方法调用分派到指定的调用处理程序。

```java
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h)
```



示例代码

```java
public class Main {
    public static void main(String[] args) {
        InvocationHandler handler = new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println(method);
                if (method.getName().equals("morning")) {
                    System.out.println("Good morning, " + args[0]);
                }
                return null;
            }
        };
        Hello hello = (Hello) Proxy.newProxyInstance(
            Hello.class.getClassLoader(), // 传入ClassLoader
            new Class[] { Hello.class }, // 传入要实现的接口
            handler); // 传入处理调用方法的InvocationHandler
        hello.morning("Bob");
    }
}

interface Hello {
    void morning(String name);
}

```

在运行期动态创建一个`interface`实例的方法如下：

1. 定义一个`InvocationHandler`实例，它负责实现接口的方法调用；
2. 通过`Proxy.newProxyInstance()`创建interface实例，它需要3个参数：
   1. 使用的`ClassLoader`，通常就是接口类的`ClassLoader`；
   2. 需要实现的接口数组，至少需要传入一个接口进去；
   3. 用来处理接口方法调用的`InvocationHandler`实例。
3. 将返回的`Object`强制转型为接口。

# 注解

注解分为内置注解和元注解



元注解：解释其他注解的注解，有以下四个

`@Target`：描述注解可以使用在什么地方

- 类或接口：ElementType.TYPE；
- 字段：ElementType.FIELD；
- 方法：ElementType.METHOD；
- 构造方法：ElementType.CONSTRUCTOR；
- 方法参数：ElementType.PARAMETER。

`@Retention`：表示注解在什么地方还有效、用于描述注解的生命周期，通常使用RUNTIME，默认为ClASS

`@Documented`：说明注解将被包含在javadoc中

`@Inherited`：说明子类可以继承父类中的该注解



自定义注解：

```java
@Target({Element.TYPE,Element.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnno{
    //注解的参数：参数类型+参数名+();
    //默认值，没有默认值则必须给参数赋值
    String name() defaule "";
    
    //value使用时可以省略
    String value();
}
```

注解本身没有任何逻辑方法，通常配合反射写逻辑方法配合注解使用。



# 日志

## 日志级别

log4j定义了8个级别的log（除去OFF和ALL，可以说分为6个级别），优先级从高到低依次为：OFF、FATAL、ERROR、WARN、INFO、DEBUG、TRACE、 ALL。

**DEBUG：** 指出细粒度信息事件对调试应用程序是非常有帮助的，主要用于开发过程中打印一些运行信息。

**INFO：** 消息在粗粒度级别上突出强调应用程序的运行过程。打印一些你感兴趣的或者重要的信息，这个可以用于生产环境中输出程序运行的一些重要信息，但是不能滥用，避免打印 　　　　过多的日志。

**WARN：** 表明会出现潜在错误的情形，有些信息不是错误信息，但是也要给程序员的一些提示。

**ERROR：** 指出虽然发生错误事件，但仍然不影响系统的继续运行。打印错误和异常信息，如果不想输出太多的日志，可以使用这个级别。

**FATAL：** 指出每个严重的错误事件将会导致应用程序的退出。这个级别比较高了。重大错误，这种级别你可以直接停止程序了。

**OFF：** 最高等级的，用于关闭所有日志记录。

## SLF4J、Log4j、Logback区别和联系

slf4j是java的一个日志门面，实现了日志框架一些通用的api，log4j和logback是具体的日志框架。

他们可以单独的使用，也可以绑定slf4j一起使用。

单独使用，分别调用框架自己的方法来输出日志信息。绑定slf4j一起使用。调用slf4j的api来输入日志信息，具体使用与底层日志框架无关（需要底层框架的配置文件）。显然不推荐单独使用日志框架。假设项目中已经使用了log4j，而我们此时加载了一个类库，而这个类库依赖另一个日志框架。这个时候我们就需要维护两个日志框架，这是一个非常麻烦的事情。而使用了slf4j就不同了，由于应用调用的抽象层的api，与底层日志框架是无关的，因此可以任意更换日志框架。

SLF4J和logback结合使用时需要提供的jar:`slf4j-api.jar,logback-classic.jar,logback-core.jar`

SLF4J和log4j结合使用时需要提供的jar:`slf4j-api.jar,slf4j-log412.jar,log4j.jar`

***slf4j-api.jar:对外提供统一的日志调用接口，该接口具体提供的调用方式和方法举例说明：\***

```java
public class Test {

　　private static final Logger logger = LoggerFactory.getLogger(Tester.class); //通过LoggerFactory获取Logger实例
　　public static void main(String[] args) {
   //接口里的统一的调用方法，各具体的日志系统都有实现这些方法
　　 logger.info("testlog: {}", "test"); 
    logger.debug("testlog: {}", "test");
    logger.error("testlog: {}", "test");
    logger.trace("testlog: {}", "test");
    logger.warn("testlog: {}", "test");
　　}
}
```

**为什么要使用SLF4J?**

-  slf4j是一个日志接口，自己没有具体实现日志系统，只提供了一组标准的调用api,这样将调用和具体的日志实现分离，使用slf4j后有利于根据自己实际的需求更换具体的日志系统，比如，之前使用的具体的日志系统为log4j,想更换为logback时，只需要删除log4j相关的jar,然后加入logback相关的jar和日志配置文件即可，而不需要改动具体的日志输出方法，试想如果没有采用这种方式，当你的系统中日志输出有成千上万条时，你要更换日志系统将是多么庞大的一项工程。如果你开发的是一个面向公众使用的组件或公共服务模块，那么一定要使用slf4的这种形式，这有利于别人在调用你的模块时保持和他系统中使用统一的日志输出。
- slf4j日志输出时可以使用{}占位符，如，logger.info("testlog: {}", "test")，而如果只使用log4j做日志输出时，只能以logger.info("testlog:"+"test")这种形式，前者要比后者在性能上更好，后者采用+连接字符串时就是new 一个String 字符串，在性能上就不如前者。



















必背会：

- [ ] String StringBuffer StringBuilder
- [ ] collection下的集合
- [ ] 多线程
- [ ] 线程休眠 Thread.sleep和TimeUnit.Second.sleep



个人简介

专业技能















































