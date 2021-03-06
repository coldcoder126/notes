##补充知识

####OpenJDK和OracleJDK

Oracle宣布以后将会同时发行两个JDK：一个是以GPLv2+CE协议下由Oracle发行的OpenJDK（本书后面章节称其为Oracle OpenJDK），另一个是在新的OTN协议下发行的传统的OracleJDK，这两个JDK共享绝大部分源码，在功能上是几乎一样的，核心差异是前者可以免费在开发、测试或生产环境中使用，但是只有半年时间的更新支持。

native方法（本地方法）：简单地讲，一个Native Method就是一个java调用非java代码的接口，来实现更底层的操作 
.java文件的一生：

1. 源码->词法分析->语法分析->语义分析->生成字节码
2. 类加载：将字节码以文件形式加载到内存->连接(校验、准备、解析)->初始化后，最终形成可以被虚拟机直接使用的Java类型的过程
3. 使用->卸载 

![jvm体系结构](C:\Users\DX\Desktop\正在编辑\images\jvm.webp)

##Java虚拟机运行时数据区

​	**程序计数器|虚拟机栈|本地方法栈|堆|方法区**

###程序计数器

可以看做是当前线程所执行的字节码的行号指示器，因此它是线程私有的且不存在内存溢出。

### 虚拟机栈

方法在执行时会创建**栈帧**，每个方法从调用到执行完成的过程就对应着一个栈帧在虚拟机栈中入栈的过程，它也是线程私有的

- 栈帧(Stack Frame)存储了方法的局部变量表、操作数栈、动态连接、和方法返回地址、额外的附加信息。（可在IDEA的debug模式中的Frames中看到）
- 非线程私有的变量使用起来就可能不是线程安全的；判断一个变量是不是线程安全，不仅要看它是不是方法中的局部变量，还要看它是否逃离了这个方法的作用范围（被return）。
- 栈内存溢出：
  - 栈帧过多，一直入栈不出栈就会导致栈溢出（例如：无限递归调用）
  - 排查cpu过多占用：ps H -eo pid,tid,%cpu | grep 进程id -> 将出问题的线程号转换为16进制 -> 在【jstack 进程号】中寻找对应的nid号 -> 定位到出现问题的代码行数
  - 排查线程死锁：jstack 

### 本地方法栈

本地方法：底层用C语言写的方法，java通过接口去调用这些方法

本地方法栈与虚拟机栈作用相似，但为虚拟机使用到的Native方法服务  

### Java堆

所有的对象实例和数组都在堆上分配内存并存放在此；是GC管理的主要区域；在虚拟机启动时创建；线程共享，非线程安全；-Xmx

堆内存溢出错误（OutOfMemoryError）：不断创建使用的实例

堆内存诊断（直接在IDEA的terminal中使用）：

​	jps工具：查看当前系统中有哪些java进程

​	jmap工具：查看某一时刻堆内存的占用情况【jmap -heap Pid】

​	jconsole工具：图形界面的，多功的检测工具，可以实现连续观测

​	jvitualvm工具：其中堆Dump功能可查看堆中对象详细信息，更详细排查

### 方法区

线程共享；用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据。

方法区内存溢出：

####运行时常量池

方法区的一部分

反编译：进入class目录 -> javap -v xxx.class

反编译后可以看到一个Constant pool ，即常量池

常量池：存在于字节码文件中。就是一张表，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型等信息

运行时常量池：当该类的.class文件被加载，它的常量池信息就会放入运行时常量池，并把里面的符号地址变为真实地址。

StringTable（串池）：在堆中；hash table结构；当需要一个String对象时，先去串池找，若没有，则把该对象加入串池，每个取值不同的String对象唯一；会发生GC

方法区的一部分；存放编译器生成的各种字面量和符号引用；受方法区内存限制

StringTable特性：

- 常量池中的字符串仅是符号，第一次用到时才变为对象
- 利用串池的机制来避免重复创建字符串对象
- 字符串变量拼接的原理是StringBuilder(1.8及以后)
- 字符串常量拼接的原理是编译期优化
- 可以使用intern方法，主动将串池中还没有的字符串对象放入串池 

```java
public class main {
    public static void main(String[] args) {
        String s1 = "a";
        String s2 = "b";
        String s3 = "ab";
        String s4 = s1 + s2;
        //过程：new StringBuilder() -> 调用其无参构造 -> append("a") -> append("b") -> toString() | toString方法new String("ab")在堆中
        System.out.println(s3 == s4); //false 因为s3在串池中而s4在堆中

        String s5 = "a"+"b"; //结果已经在编译期确定为"ab",故直接到串池中找"ab",由于串池中已有，故不用创建新对象
        System.out.println(s4 == s5); //true
    }
}

```

StringTable性能调优：

由于StringTable是哈希表，所以可以通过将StringTableSize值变大以减少哈希冲突从而提升性能。

考虑将字符串对象加入到StringTable以减少重复对象的数量

####直接内存（Direct Memory）

（它并不是虚拟机运行时数据区的一部分）文件读取的过程中，要先把文件读到操作系统内存，然后再复制到JVM内存，需要两次操作；而直接内存就是开辟一块操作系统的内存，同时JVM也可以访问。

常见于NIO操作时，用于数据缓冲区；

读写性能较高，但分配回收成本较高；

不受JVM内存回收管理。

使用Unsafe.freeMemory清理释放

##Java对象的创建

1. 虚拟机遇到一条new指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有则执行类加载过程
2. 类加载通过后，分配内存
3. 将分配到的内存空间（除对象头外）初始化为零值->对对象进行必要的设置->将必要信息存放在对象头之中
4. 执行<init>()方法，把对象按照程序员的意愿初始化

#### 对象的内存布局

在堆内存中，对象：{对象头、实例数据、对齐填充}

对象在内存中的存储
|-对象头   
	|-自身运行时数据：哈希码、GC分代年龄、锁状态标志等  
	|-类型指针  
|-实例数据  
|-对齐填充 

##垃圾回收GC(Garbage Collection)

程序计数器、虚拟机栈和本地方法栈生命周期线程相同，故GC主要关注的是Java堆和方法区的内存
主要要完成的三个问题

- 哪些内存需要回收
- 什么时候回收
- 如何回收

1. 要明白哪些对象需要回收，首先需要弄清楚准备回收的对象它“死”了吗？如何判断呢？

   1. 引用计数算法：给每个对象添加一个引用计数器，有一个地方引用它时，数字加1，当这个数字为0的时候，代表该对象可回收。但无法解决对象间循环引用的问题，故已淘汰。

   2. 可达性分析算法：通过【GC Roots】对象作为起始点，从这些节点向下遍历了，当GC Roots 到这个对象不可达时，代表这个对象不可用。不可达之后还要经过两次标记才能宣布此对象已死。

      >何为GC Roots？
      >在java中，可作为GC Roots的对象包括下面几种：
      >
      >- 虚拟机栈中引用的对象
      >- 方法区中静态属性引用的对象
      >- 方法区中常量引用的对象  

>- Native方法引用的对象
>  两总算法都用到了引用，那么如何定义引用？
>
>  >引用分为强引用、软引用、弱引用和虚引用四种

		> GC Roots如何查看：
		>
		> jps查看PID -> jmap -dump:format=b,live,file=1.bin PID -> 使用Eclipse Memory Analyzer打开 -> java basic  -> gc_roots

- 强引用：普遍存在的类似```A a = new A(); ```这类引用，GC不会回收掉强引用对象
- 软引用：描述一些还有用但非必须的对象，系统发生内存溢出异常之前，才会回收。可用`SoftReference`类实现；恰当使用软引用可以节省内存。软引用非常适合做缓存。
- 弱引用：比软引用更弱，若引用关联的对象只能生存到下一次垃圾收集之前，下次GC一定回收。可用`WeakReference`类实现。可以解决某些地方内存泄漏的问题，如ThreadLocal
- 虚引用：最弱，为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。可用`PhantomReference`实现。 可以用于管理堆外内存(DirectByteBuffer)   

引用队列：清理软引用对象本身（引用也是一个对象）

```java
#若想将A -引用-> B 变成 A -软/弱引用-> B 则要A引用Soft/WeakReference对象(ref),再通过ref引用B
ReferrnceQueue<byte[]> queue = new ReferenceQueue<>();	//引用队列
SoftReference<byte[]> ref = new SoftReference<>(new byte[],queue);	//当ref关联的对象被回收时，该引用会自动加入到队列	
```

###垃圾回收算法

标记-清除算法：速度快但是容易产生内存碎片

标记-整理算法：将标记清除所产生的内存碎片整理到一起，需要进行对象的移动，速度较慢

复制算法：将FORM 区中存活的对象复制到TO区域中，并在这个过程中实现内存的整理，然后两个区域交换角色，但是要用到两倍的空间。（适用于每次垃圾收集都有大批对象死去，只有少量存活）

分代收集算法：根据对象存货周期的不同将内存划分为几块(这些区域划分仅仅是一部分垃圾收集器的共同特性或者说设计风格而已，而非固有内存布局)

堆内存：{新生代：{伊甸园，幸存区FROM，幸存区TO}，老年代}

老年代：长时间使用的对象

新生代：短时间使用的对象，比老年代空间小

流程：新创建的对象会被放到【伊甸园】放不下时会触发一次【Minor GC】，将仍然存活的对象复制到【幸存区From】中，并将幸存对象的寿命+1，下一次GC会将【伊甸园】仍然存活的对象放到【幸存区To】，【幸存区From】中的存活对象寿命+1并复制到【幸存区To】，【幸存区To】腾出来，两个幸存区交换位置，循环往复；新对象继续放入【伊甸园】，满了之后触发【Minor GC】，如上述操作，除此之外【幸存区To】中仍然存活的寿命继续+1，【幸存区From】和【幸存区To】交换位置，若一个对象寿命超过某个阈值（默认15）则进入老年代，若【幸存区To】满了则将其中所有对象移入老年代；老年代空间都不够的话，触发一次Full GC，若触发Full GC之后发现空间依然不够用，就会产生`OutOfMemoryError`(**todo**:从新生代老年代都回收一遍 只要老年代的连续空间大于新生代的对象总大小或者平均晋升大小就会进入Full GC )

- 新生代：老年代默认为1：2
- 新生代中各空间占用比例，Eden：From：To默认为8：1：1
- Minor GC 更频繁，Full GC更费时(是Minor GC的10倍)
- // From区和To区，名称不是固定的，每次GC之后都会互换

minor gc会引发stop the world ，暂停用户的线程，等垃圾回收结束，用户线程恢复运行

| 含义                    | 参数                           |
| ----------------------- | ------------------------------ |
| 堆初初始大小            | -Mms                           |
| 堆最大大小              | -Xmx                           |
| 新生代大小              | -Xmn                           |
| 幸存区比例（动态）      | -XX:InitialSurvivorRatio=ratio |
| 百度一下/有截图然后填好 | 多个命令之间用空格隔开         |
|                         |                                |
|                         |                                |

大对象直接进入老年代：新生代即时是垃圾回收之后也容不下新对象，若此时老年代空间足够，则该直接晋升到老年代

安全点（Safepoint）:一个特定的位置，GC时只有在安全点才能暂停

安全区域（Safe Region）：安全区域是指在一段代码片段之中，引用关系不会发生变化。在这个区域中任意地方开始GC都是安全的，可看做扩展了的安全点

### 垃圾收集器：

垃圾收集器是内存回收的具体实现；HotSpot虚拟机中包含7种收集器，组合使用；基本类型包括以下三种：

串行：单线程，回收垃圾时必须暂停所有的工作线程；

吞吐量优先：多线程，适合堆内存较大的环境且多核CPU支持；让Stop the world发生的次数少 

响应时间优先：多线程，适合堆内存较大的环境且多核CPU支持尽可能让单次Stop the word 时间最短

- Serial收集器:工作在新生代；使用复制算法；简单高效，适合堆内存较小的环境，Client模式下的默认的新生代收集器
- Serial Old收集器：工作在老年代；标记-整理算法
- ParNew 收集器：Serial收集器的多线程版本；可与CMS配合工作
- Parallel Scavenge收集器：新生代；复制算法；并行的多线程收集器；尽可能达到一个可控制的【吞吐量】（吞吐量=运行用户代码时间/(运行用户代码时间+垃圾收集时间)）；适合在后台运算而不需要太多交互的任务；可配置自适应调节
- Parallel Old收集器：是Parallel Scavenge收集器的老年代版本；多线程；标记-整理算法
- CMS(Concurrent Mark Sweep)收集器：以达到最短回收停顿时间为目标的收集器；标记-清除算法
- **G1收集器**：同时注重吞吐量和低延迟；超大堆内存，将堆划分为多个region；其回收得到的空间是连续的；从整体上来看是基于"标记-整理"算法，从局部（两个region）上来看是基于"复制"算法。

####G1收集器

G1把连续的Java堆划分为多个大小相等的独立区域，每个区域取值范围为1M~32M且为2的N次幂。每个区域都可以根据需要去扮演新生代或老年代空间，收集器对扮演不同角色的区域采用不同的策略处理。进行垃圾回收的标准不再是它属于哪个分代，而是哪块内存中存放的垃圾数量最多，回收收益最大

## 垃圾回收调优

### 新生代调优

新生代的特点：

- 所有的new 操作的内存分配非常廉价
- 死亡对象的回收代价是零
- 大部分对象用过即死
- Minor GC的时间远远低于Full GC 
- 晋升域值配置得当，让长时间存活对象尽快晋升`-XX:MaxTenuringThreshod=threshold` 打印各age占用的空间：`-XX:+PrintTenuringDistribution` 

新生代过小会导致过于频繁的Minor GC；新生代过大会导致只有Full GC ;建议新生代占用总堆的1/4~1/2

新生代要能容纳所有【并发量*一次请求相应的数据量】

​	幸存区要能保留【当前活跃对象+需要晋升的对象】

###老年代调优

CMS为例：

- 老年代内存越大越好

- 先尝试不做调优，如果没有Full GC 则已经OK，否则先尝试调优新生代

- 观察发生Full GC时老年代内存占用，将老年代内存预设调大1/4~1/3`-XX:CMSInitiatingOccupancyFraction=percent`

  

## 类加载器 ClassLoader

负责加载class文件，在class文件内开头有特定的文件标志`cafe babe`，将class文件字节码内容加载到内存中，并将这些内容转换成方法区中的运行时数据结构。ClassLoader只负责class文件的加载，至于它是否可以运行，则由Execution Engine决定。

<img src="C:\Users\DX\Desktop\正在编辑\images\classloader.png" alt="类加载过程" style="zoom:50%;" />

`Car.class`经过`ClassLoader`加载并初始化到JVM中成为`Car Class`模板，由此模板可以创建多个实例

**加载器的种类：**

虚拟机自带的加载器

- 启动类加载器(Bootstrap)，C++实现
- 扩展加类载器(Extension) ，系统自带的后来扩展的类
- 应用程序类加载器(AppClassLoader)，加载当前应用的classpath所有类，即自定义的类

用户自定义加载器：

`java.lang.ClassLoader`的子类，用户可以定制类的加载方式

> AppClassLoader 继承自 Extension 继承自 Bootstrap

装完jdk就能使用基本的类型是因为Bootstrap加载器将`rt.jar`加载到了JVM中，`rt.jar`中有系统自带的类

**查看自己的加载器：**

`obj.getClass().getClassLoader()`会返回加载器名称，`Bootstrap`类加载器返回null

双亲委派机制：

> 双亲委派：如果一个类加载器收到了加载某个类的请求,则该类加载器并不会去加载该类,而是把这个请求委派给父类加载器,每一个层次的类加载器都是如此,因此所有的类加载请求最终都会传送到顶端的启动类加载器;只有当父类加载器在其搜索范围内无法找到所需的类,并将该结果反馈给子类加载器,子类加载器会尝试去自己加载。
>
> 这样保证了使用不同的类加载器最终得到的都是同一个Object对象

沙箱安全机制



### 类文件结构

### 字节码指令

### 编译期处理

### 类加载阶段

类加载器

在类加载的第一阶段“加载”过程中，需要通过一个类的全限定名来获取定义此类的二进制字节流，完成这个动作的代码块就是 **类加载器**。这一动作是放在 Java 虚拟机外部去实现的，以便让应用程序自己决定如何获取所需的类。

收集器基本都采用分代收集算法，java堆中可以分为新生代和老年代

## JVM面试题

1. 性能调优参数：

   -Xss：

   -Xmx：

   -Xms：

2. 占位