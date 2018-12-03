# 一、深入理解虚拟机之Java内存区域：
## 1、介绍下Java内存区域（运行时数据区）
![](https://mmbiz.qpic.cn/mmbiz_png/hvUCbRic69sD4O5ZOeBZiaRDJq1VA8wFqz2BW8LPJU4Cw4hSyZZK0CWJibof9MicmCaeb3xmVDTVfJqJYgC3yYzGOw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
* Java虚拟机在执行Java程序的过程中会把它管理的内存划分成若干个不同的数据区域:
    * 线程共享区
        * 1）方法区：用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等等
            * 运行时常量池：运行时常量池是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有常量池信息（用于存放编译期生成的各种字面量和符号引用）
        * 2）堆：用于存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。
            * 是垃圾收集器管理的主要区域，因此也被称作GC堆（Garbage Collected Heap）
    * 线程独占区
        * 3）程序计数器：是当前线程所执行的字节码的行号指示器。 
            * 【字节码解释器工作时通过改变这个计数器的值来选取下一条需要执行的字节码指令。为了能在线程切换后能恢复到正确的执行位置，所以每条线程都得有一个】
        * 4）Java虚拟机栈：即常说的栈内存。Java虚拟机栈描述的是Java方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧，用于存储局部变量表、操作数栈、动态链接、方法出入口等信息。 
            * 局部变量表主要存放了编译器可知的各种数据类型、对象引用
        * 5）本地方法栈：类似Java虚拟机栈。虚拟机栈为虚拟机执行Java方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的Native方法服务。
        
## 2、对象的访问定位的两种方式
* Java程序通过栈上的引用数据来操作堆上的具体对象。目前主流的对象访问方式有：
    * 使用句柄：
        * 如果使用句柄的话，那么Java堆中将会划分出一块内存来作为句柄池，引用中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息
        ![](https://mmbiz.qpic.cn/mmbiz_png/hvUCbRic69sD4O5ZOeBZiaRDJq1VA8wFqzeVX5DSk1nAnbuwE1UFcfiaiaLoN4nFSXnyDricrG49CadGAmOgpwaVx8Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
    * 直接指针：
        * 如果使用直接指针访问，那么Java堆对象的布局中就必须考虑如何防止访问类型数据的相关信息，reference中存储的直接就是对象的地址
        ![](https://mmbiz.qpic.cn/mmbiz_png/hvUCbRic69sD4O5ZOeBZiaRDJq1VA8wFqzfP0g2FMdicNQBOa7FKQeBiasS5ic9DgaNsMyKqsjZncicx5kRy1cORs4Qg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
    * 各自的优点：
        * 使用句柄来访问的最大好处是引用中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而引用本身不需要修改。
        * 使用直接指针访问方式最大的好处就是速度快，它节省了一次指针定位的时间开销。

# 二、深入理解虚拟机之垃圾回收

## 1、如何判断对象是否死亡？——引用计数法、可达性分析算法
* 有两种算法判断对象是否死亡：
    * 引用计数法：给对象中添加一个引用计数器，每当有一个地方引用它，计数器就加1；当引用失效，计数器就减1；任何时候计数器为0的对象就是不可能再被使用的。
        * 这个方法实现简单，效率高，但是目前主流的虚拟机中并没有选择这个算法来管理内存，其最主要的原因是它很难解决对象之间相互循环引用的问题。
    * 可达性分析算法：通过一系列的称为 “GC Roots” 的对象作为起点，从这些节点开始向下搜索，节点所走过的路径称为引用链，当一个对象到GC Roots没有任何引用链相连的话，则证明此对象是不可用的。
    ![](https://mmbiz.qpic.cn/mmbiz_png/hvUCbRic69sDOMkMYwE1nhEJFZN46xicicavGRIQiax8E4PaRW4fQxgEeiax0PhqkKD24Y6fG8Ao5WBD0ibVWCoImU7A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 2、简单的介绍一下强引用、软引用、弱引用、虚引用（虚引用与软引用和弱引用的区别、使用软引用能带来的好处）。
* 1) 强引用：我们使用的大部分引用实际上都是强引用，这是使用最普遍的引用。如果一个对象具有强引用，垃圾回收器绝不会回收它。
    * 当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足问题。
* 2）软引用（通过SoftReference类来实现软引用）：如果一个对象只具有软引用，在内存空间足够时，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。
    * 只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。
* 3) 弱引用（通过WeakReference类来实现弱引用）：如果一个对象只具有弱引用，那么当垃圾回收器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。
    * 不过，由于垃圾回收器是一个优先级很低的线程， 因此不一定会很快发现那些只具有弱引用的对象。
* 4）虚引用（PhantomReference）：无法通过虚引用来取得一个对象实例。为对象设置虚引用的唯一目的就是能在这个对象被收集器回收时收到一个系统通知
```java
Object obj = new Object();
SoftReference<Object> sf = new SoftReference<Object>(obj);
obj = null;
sf.get();//有时候会返回null
这时候sf是对obj的一个软引用，通过sf.get()方法可以取到这个对象，当然，当这个对象被标记为需要回收的对象时，则返回null；
```
## 3、垃圾收集有哪些算法，各自的特点？
* 1） 标记-清除算法：首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象。
    * 它是最基础的收集算法，存在两大问题：
        * 1：效率问题
        * 2：空间问题（标记清除后会产生大量不连续的碎片）
* 2) 复制算法：为了解决效率问题，“复制”收集算法出现了。它可以将内存分为大小相同的两块，每次使用其中的一块。当这一块的内存使用完后，就将还存活的对象复制到另一块去，然后再把使用的空间一次清理掉。
    * 问题：代价是内存缩小为原来的一半
* 3）标记-整理算法：根据老年代的特点特出的一种标记算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象回收，而是让所有存活的对象向一端移动，然后直接清理掉端边界以外的内存。
* 4） 分代收集算法：根据对象存活周期的不同将内存分为几块。
    * 优点：一般将java堆分为新生代和老年代，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。
        * 比如在新生代中，每次收集都会有大量对象死去，所以可以选择复制算法，只需要付出少量对象的复制成本就可以完成每次垃圾收集。
        * 而老年代的对象存活几率是比较高的所以我们可以选择“标记-清理”或“标记-整理”算法进行垃圾收集。
## 4、HotSpot为什么要分为新生代和老年代？
* 见上分代收集算法的优点
## 5、常见的垃圾回收器有那些？【未】
* 1）Serial收集器：这是一个单线程收集器。
    * 缺点：它只会使用一条垃圾收集线程去完成垃圾收集工作，且它在进行垃圾收集工作的时候必须暂停其他所有的工作线程（ "Stop The World" ），直到它收集结束
    * 优点：简单高效，因为没有线程交互的开销
![](https://mmbiz.qpic.cn/mmbiz_png/hvUCbRic69sDOMkMYwE1nhEJFZN46xicicaLUCFF3pUm6UObRx0nJXvibhjU5uiawbrficoq1a0ZT7YwNI37vOaK8Cdg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
* 2） ParNew收集器：Serial收集器的多线程版本，除了使用多线程进行垃圾收集外，其余行为（控制参数、收集算法、回收策略等等）和Serial收集器完全一样
## 6、介绍一下CMS,G1收集器。【未】

## 7、Minor Gc和Full GC 有什么不同呢？
* 新生代GC（Minor GC）:指发生新生代的的垃圾收集动作，Minor GC非常频繁，回收速度一般也比较快。
* 老年代GC（Major GC/Full GC）:指发生在老年代的GC，出现了Full GC经常会伴随至少一次的Minor GC（并非绝对），Full GC的速度一般会比Minor GC的慢10倍以上。

## 8、内存分配与回收策略
* 1) 对象优先在Eden区分配
* 2) 大对象直接进入老年代
* 3) 长期存活的对象将进入老年代
* 4) 动态对象年龄判定：为了更好的适应不同程序的内存情况，虚拟机不是永远要求对象年龄必须达到了某个值才能进入老年代，如果Survivor 空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无需达到要求的年龄。


# 五、深入理解Java虚拟机之虚拟机类加载机制
## 1、简单说说类加载过程，里面执行了哪些操作？

## 2、对类加载器有了解吗？

## 3、什么是双亲委派模型？

## 4、双亲委派模型的工作过程以及使用它的好处。