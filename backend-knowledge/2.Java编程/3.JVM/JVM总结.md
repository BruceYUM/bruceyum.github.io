# JVM总结

[toc]

## 一、自动内存管理

### 1、运行时数据区

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/5778d113-8e13-4c53-b5bf-801e58080b97.png) 

- 程序计数器：程序计数器用于指示当前线程下一条要执行的指令的地址；每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响；如果线程正在执行的是一个 Java 方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是 Native 方法，这个计数器值则为 Undefined。此内存区域是唯一一个在 Java 虚拟机规范中没有规定任何 OutOfMemoryError 情况的区域。

- JAVA 虚拟机栈：每条线程所私有，虚拟机栈由栈帧构成，每个栈帧包括：局部变量表、操作数栈、动态链接和返回地址等。线程每调用一个 Method；就会在虚拟机栈中创建一个栈帧，方法执行完该栈帧出栈动态的创建与销毁。

    局部变量表存放了编译期可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference 类型，它不等同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）和 returnAddress 类型（指向了一条字节码指令的地址）。<br>
    该区域可能存在的内存溢出有两种情况：启用过多的线程导致没有足够的空间创建新的线程；方法调用的深度过深，不能创建更多的栈帧。

- 本地方法区：本地方法栈（Native Method Stack）与虚拟机栈所发挥的作用是非常相似的，它们之间的区别不过是虚拟机栈为虚拟机执行 Java 方法（也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务，也是线程私有。

- JAVA 堆：该区域是所有线程共享的，保存着对象；该区域是垃圾收集器管理的主要区域，可以进一步划分为Eden区域、From Survivor区域、To Survivor区域以及老年代；如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError异常。

- 方法区：也是所有线程共享，用于存储已被虚拟机加载的类信息、常量池、静态变量、即时编译器编译后的代码等数据。在HotSpot中也可以认为方法区等价于【永久代】，这是因为HotSpot在实现的时候用永久代替代方法区，以是的GC更好的释放方法区内存。

    常量池：用于存放编译期生成的各种字面量和符号引用

注意：直接内存使用Native函数库直接分配堆外内存，然后通过一个存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域。但是这部分内存也被频繁地使用，而且也可能导致OutOfMemoryError异常出现


### 2、对象的创建

- 类加载：虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有，那必须先执行相应的类加载过程

- 分配空间：为对象分配空间的任务等同于把一块确定大小的内存从 Java 堆中划分出来。根据 JAVA GC 算法的不同，有不同的空间分配方法。

    - 指针碰撞（Bump the Pointer）：如果 GC 采用标记-压缩或者复制算法，则 GC 执行之后，已分配空间和未分配空间是连续的中间用一个特殊的指针标记，然后从未分配空间划出一部分分配给新的对象，挪动标记指针；
    
    - 空闲列表：GC采用标记-清楚除算法则已分配空间和未分配空间是交替的，此时 JVM 需要维护一张空间分配表，用于指示哪些空间已分配，哪些未分配；然后找到一块足够大的空间分配给新的对象。
    
    - 对象内存分配的线程安全问题：例如线程A读取分界点指示器，分给空间还未修改分界点指示器，线程 B 使用同样的分界点指示器划分空间则导致线程 A 和线程 B 创建的对象空间重叠；为了解决这个问题有两种解决方法：第一种是采用 CAS 方式；第二种是采用 ThreadLocal 方式，即每个线程在 Java 堆中预先分配一小块内存，称为本地线程分配缓冲（Thread Local Allocation Buffer,TLAB）

- 初始化零值：除了对象头以外的其他空间初始化为零值，基本类型为0，引用类型为null，这一步保证在Java代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。

- 设置对象头：这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的GC分代年龄等信息。这些信息存放在对象的对象头（Object Header）之中。根据虚拟机当前的运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。

- 初始化赋值：执行new指令之后会接着执行<init>方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。

### 3、对象的内存布局

- 对象头：由两部分信息组成
    - 第一部分保存对象自身运行时的数据，根据对象自身所处的状态不同保存的内容也不同，包括：哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等，这部分数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32bit和64bit，官方称它为“Mark Word”。
    
        ![img](https://note.youdao.com/yws/api/personal/file/0E461A944D5A484A870A091100645E6E?method=getImage&version=1316&cstk=4sbHpvdq) 
    
    - 第二部分：指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。如果对象是一个Java数组，那在对象头中还必须有一块用于记录数组长度的数据。

- 实例数据：对象真正存储的有效信息，也是在程序代码中所定义的各种类型的字段内容。无论是从父类继承下来的，还是在子类中定义的，都需要记录起来。

- 对齐填充：并不是必然存在的，也没有特别的含义，它仅仅起着占位符的作用。由于HotSpot VM的自动内存管理系统要求对象起始地址必须是8字节的整数倍


实战OOM见代码：


## 二、垃圾收集器

### 1、可达性分析

**1. GC Roots**
- GC Roots：虚拟机栈（栈帧中的本地变量表）中引用的对象。

    方法区中类静态属性引用的对象。
    
    方法区中常量引用的对象。
    
    本地方法栈中 JNI（即一般说的 Native 方法）引用的对象。

- 从 GC Roots 开始向下搜索，搜索所走过的路径称为引用链（Reference Chain），当一个对象到 GC Roots 没有任何引用链相连时，则证明此对象是不可达的。当对象不可达的时候我们认为它可以被回收（当然还会与 finalize() 函数相关；当程序在 finalize() 函数中将当前对象重新引用，则该对象不会被回收）

**2. 找到 GC Roots**

问题：我们说 GC Roots 存在于方法区常量引用，静态变量引用以及虚拟机栈上局部变量引用；可是这些空间比较大，如果需要逐个扫描找出那些是引用需要耗费很多时间；而且当进行 GC 的时候会造成“全局停顿”问题，所以希望尽可能的减少时间消耗。

那么在 HotSpot 中虚拟机直接指导那些地方存的是引用那些地方存的是基本数据类型；在 HotSpot 的实现中，是使用一组称为 OopMap 的数据结构来达到这个目的的，在类加载完成的时候，HotSpot 就把对象内什么偏移量上是什么类型的数据计算出来，在 JIT 编译过程中，也会在特定的位置记录下栈和寄存器中哪些位置是引用。

**3. 安全点**

- 如何利用 OopMap 确定哪里是引用类型：OopMap { ebx= Oop[16] = Oop off=142} 是一条 OopMap，它指示在 EBX 寄存器和栈中偏移量为 16 的内存区域中各有一个普通对象指针（Ordinary Object Pointer）的引用，有效范围为从 call 指令开始直到（指令流的起始位置）+142（OopMap记录的偏移）。

- 在哪里添加 OopMap：OopMap 的内容可能是会变化的，比如当前应用指向对象 A，随着程序的执行可能指向对象 B，那么 OopMap 的内容就会变化；我们不可能为每一条指令添加一个 OopMap 记录。那么我们在什么位置添加 OopMap 记录呢？

    在 HotSpot 中我们只在安全点添加 OopMap 指令；那什么是安全点呢？安全点是代码中一些“特定位置”，程序只有执行到这些位置才能够停顿下来 GC；安全点的选定基本上是以程序“是否具有让程序长时间执行的特征”为标准进行选定的。“长时间执行”的最明显特征就是指令序列复用，例如方法调用、循环跳转、异常跳转等，所以具有这些功能的指令才会产生安全点。那么我们可以在这些具有长时间执行特征的指令集前设置为安全点；所有线程在 GC 全局停顿是中断在安全点。OopMap 也添加在安全点。

- 如何让所有线程在发生 GC 的时候中断在安全点上：

    - 抢先式中断：在 GC 发生时，首先把所有线程全部中断，如果发现有线程中断的地方不在安全点上，就恢复线程，让它“跑”到安全点上
    
    - 主动式中断：当 GC 需要中断线程的时候，不直接对线程操作，仅仅简单地设置一个标志，各个线程执行时主动去轮询这个标志，发现中断标志为真时就自己中断挂起


**4. 安全区域**

为什么会有安全区域：当 GC 是能够保证执行中的程序找到合适的安全点中断；但是处于 WAIT 和 BLOCKED 状态的程序呢？程序可能无法响应中断请求执行到安全点；安全区域是指在一段代码片段之中，引用关系不会发生变化。在这个区域中的任意地方开始 GC 都是安全的。

在线程执行到 Safe Region 中的代码时，首先标识自己已经进入了 Safe Region，那样，当在这段时间里 JVM 要发起 GC 时，就不用管标识自己为 Safe Region 状态的线程了。在线程要离开 Safe Region 时，它要检查系统是否已经完成了根节点枚举（或者是整个 GC 过程），如果完成了，那线程就继续执行，否则它就必须等待直到收到可以安全离开 Safe Region 的信号为止。


### 2、引用类型

- 强引用（Strong Reference）：类似 Object obj=new Object（） 这类的引用，只要强引用还存在，垃圾收集器永远不会回收掉被引用的对象。

- 软引用（Soft Reference）：软引用是用来描述一些还有用但并非必需的对象。对于软引用关联着的对象，在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之中进行第二次回收。如果这次回收还没有足够的内存，才会抛出内存溢出异常。在 JDK 1.2 之后，提供了 SoftReference 类来实现软引用。

- 弱引用（Weak Reference）：弱引用也是用来描述非必需对象的，但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发生之前。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。在 JDK 1.2 之后，提供了 WeakReference 类来实现弱引用。

- 虚引用（Phantom Reference）：最弱的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知


### 3、回收方法区（永久代）

Java 虚拟机规范中确实说过可以不要求虚拟机在方法区实现垃圾收集，而且在方法区中进行垃圾收集的“性价比”一般比较低：在堆中，尤其是在新生代中，常规应用进行一次垃圾收集一般可以回收 70%～95% 的空间，而永久代的垃圾收集效率远低于此。永久代的垃圾收集主要回收两部分内容：废弃常量和无用的类。

- 废弃常量：在系统中若没有任何对象引用该常量，那么该常量就是废弃常量

- 无用类：该类所有的实例都已经被回收，也就是 Java 堆中不存在该类的任何实例。加载该类的 ClassLoader 已经被回收。该类对应的 java.lang.Class 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

在大量使用反射、动态代理、CGLib 等 ByteCode 框架、动态生成 JSP 以及 OSGi 这类频繁自定义 ClassLoader 的场景都需要虚拟机具备类卸载的功能，以保证永久代不会溢出。


### 4、垃圾收集算法

**1. 标记 - 清除**

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/005b481b-502b-4e3f-985d-d043c2b330aa.png)<br>

在标记阶段，程序会检查每个对象是否为活动对象，如果是活动对象，则程序会在对象头部打上标记。

在清除阶段，会进行对象回收并取消标志位，另外，还会判断回收后的分块与前一个空闲分块是否连续，若连续，会合并这两个分块。回收对象就是把对象作为分块，连接到被称为 “空闲链表” 的单向链表，之后进行分配时只需要遍历这个空闲链表，就可以找到分块。

在分配时，程序会搜索空闲链表寻找空间大于等于新对象大小 size 的块 block。如果它找到的块等于 size，会直接返回这个分块；如果找到的块大于 size，会将块分割成大小为 size 与 (block - size) 的两部分，返回大小为 size 的分块，并把大小为 (block - size) 的块返回给空闲链表。

不足：

- 标记和清除过程效率都不高；
- 会产生大量不连续的内存碎片，导致无法给大对象分配内存。

**2. 标记 - 整理**

![img](https://note.youdao.com/yws/api/personal/file/AFEFA9051C784372A8B6C6B7FEAC07D7?method=getImage&version=991&cstk=4sbHpvdq)<br>

让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

优点:

- 不会产生内存碎片

不足:

- 需要移动大量对象，处理效率比较低。

**3. 复制**

![img](https://note.youdao.com/yws/api/personal/file/6746D907C9994F9498B877FF5726B42C?method=getImage&version=942&cstk=4sbHpvdq)<br>

将内存划分为大小相等的两块，每次只使用其中一块，当这一块内存用完了就将还存活的对象复制到另一块上面，然后再把使用过的内存空间进行一次清理。

主要不足是只使用了内存的一半。

现在的商业虚拟机都采用这种收集算法回收新生代，但是并不是划分为大小相等的两块，而是一块较大的 Eden 空间和两块较小的 Survivor 空间，每次使用 Eden 和其中一块 Survivor。在回收时，将 Eden 和 Survivor 中还存活着的对象全部复制到另一块 Survivor 上，最后清理 Eden 和使用过的那一块 Survivor。

HotSpot 虚拟机的 Eden 和 Survivor 大小比例默认为 8:1，保证了内存的利用率达到 90%。如果每次回收有多于 10% 的对象存活，那么一块 Survivor 就不够用了，此时需要依赖于老年代进行空间分配担保，也就是借用老年代的空间存储放不下的对象。

**4. 分代收集**

现在的商业虚拟机采用分代收集算法，它根据对象存活周期将内存划分为几块，不同块采用适当的收集算法。

一般将堆分为新生代和老年代。

- 新生代使用：复制算法
- 老年代使用：标记 - 清除 或者 标记 - 整理 算法

 

### 5、垃圾收集器

![img](https://note.youdao.com/yws/api/personal/file/666D0722153642349CBC99BF8536DB9F?method=getImage&version=983&cstk=4sbHpvdq)<br>

以上是 HotSpot 虚拟机中的 7 个垃圾收集器，连线表示垃圾收集器可以配合使用。

- 单线程与多线程：单线程指的是垃圾收集器只使用一个线程，而多线程使用多个线程；
- 串行与并行：串行指的是垃圾收集器与用户程序交替执行，这意味着在执行垃圾收集的时候需要停顿用户程序；并行指的是垃圾收集器和用户程序同时执行。除了 CMS 和 G1 之外，其它垃圾收集器都是以串行的方式执行。

**1. Serial 收集器**

![img](https://note.youdao.com/yws/api/personal/file/EFD9B792AEFA4F92AFFA92DD7843D124?method=getImage&version=688&cstk=4sbHpvdq)<br>

Serial 翻译为串行，也就是说它以串行的方式执行。

它是单线程的收集器，只会使用一个线程进行垃圾收集工作。

它的优点是简单高效，在单个 CPU 环境下，由于没有线程交互的开销，因此拥有最高的单线程收集效率。

它是 Client 场景下的默认新生代收集器，因为在该场景下内存一般来说不会很大。它收集一两百兆垃圾的停顿时间可以控制在一百多毫秒以内，只要不是太频繁，这点停顿时间是可以接受的。


**2. ParNew 收集器**

![img](https://note.youdao.com/yws/api/personal/file/E241B09FA0D54FD0978159799D7C60A2?method=getImage&version=857&cstk=4sbHpvdq)<br>

它是 Serial 收集器的多线程版本。

它是 Server 场景下默认的新生代收集器，除了性能原因外，主要是因为除了 Serial 收集器，只有它能与 CMS 收集器配合使用。

**3. Parallel Scavenge 收集器**

与 ParNew 一样是多线程收集器。

其它收集器目标是尽可能缩短垃圾收集时用户线程的停顿时间，而它的目标是达到一个可控制的吞吐量，因此它被称为“吞吐量优先”收集器。这里的吞吐量指 CPU 用于运行用户程序的时间占总时间的比值。

停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户体验。而高吞吐量则可以高效率地利用 CPU 时间，尽快完成程序的运算任务，适合在后台运算而不需要太多交互的任务。

缩短停顿时间是以牺牲吞吐量和新生代空间来换取的：新生代空间变小，垃圾回收变得频繁，导致吞吐量下降。

可以通过一个开关参数打开 GC 自适应的调节策略（GC Ergonomics），就不需要手工指定新生代的大小（-Xmn）、Eden 和 Survivor 区的比例、晋升老年代对象年龄等细节参数了。虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量。


**4. Serial Old 收集器**

![img](https://note.youdao.com/yws/api/personal/sync?method=download&fileId=2260EA35AB2B4A5DA6E626B6FB8F6ABD&version=618&cstk=4sbHpvdq)<br>

是 Serial 收集器的老年代版本，也是给 Client 场景下的虚拟机使用。如果用在 Server 场景下，它有两大用途：

- 在 JDK 1.5 以及之前版本（Parallel Old 诞生以前）中与 Parallel Scavenge 收集器搭配使用。
- 作为 CMS 收集器的后备预案，在并发收集发生 Concurrent Mode Failure 时使用。


**5. Parallel Old 收集器**

![img](https://note.youdao.com/yws/api/personal/file/B1A55BAD782E4480BA101D4444BB38DF?method=getImage&version=699&cstk=4sbHpvdq)<br>

是 Parallel Scavenge 收集器的老年代版本。

在注重吞吐量以及 CPU 资源敏感的场合，都可以优先考虑 Parallel Scavenge 加 Parallel Old 收集器。


**6. CMS 收集器**

CMS 以获取最短的停顿时间为目标，它使用与对用户交互要求高的环境下，注重服务器的响应速度。CMS 运用于老年代，主要分为以下4个步骤：

- 初始标记：需要全局停顿，只是为了找出 GC Root 相应用的第一个对象

- 并发标记：与用户线程交替并发执行，完成标记工作

- 重新标记：需要全局停顿，修正并发标记的错误

- 并发收集：并发清除未存活对象。

    ![img](https://note.youdao.com/yws/api/personal/file/45040AC749CF4BBB9C0CC94209A84BD6?method=getImage&version=795&cstk=4sbHpvdq) 

CMS的缺点：

- 吞吐量低：虽然 CMS 几乎没有让用户线程停顿，但是 CMS 与用于线程并发执行的时候，因为占用了一部分线程（或者说 CPU 资源）而导致应用程序变慢，总吞吐量会降低。这对于 CPU 资源敏感的程序依然很难接受

- CMS 无法处理浮动垃圾：因为 CMS 与用户线程并发执行，那么在 CMS 清理的过程中用户线程会产生新的垃圾，而这部分垃圾只能留到下一次收集。并且我们需要为这部分新的垃圾留出足够的空间装下用户线程产生的浮动空间；这导致 CMS 不能像 Parallel Old 等一样，可以等到老年代满了之后在收集。在 JDK1.6 中 CMS 的启动阈值是 92%；但是如果预留的空间仍然不够使用，就会导致 CMS 失败，从而启用备用方案 Serial Old，仍然会导致全局停顿。

- CMS 采用标记清除算法，会导致内存碎片；如果大量的内存碎片可能导致虽然后足够的剩余空间，却无法找到足够大的连续空间分配给一个大对象，只是提前触发 Full GC。

    CMS 收集器提供了一个 -XX：+UseCMSCompactAtFullCollection 开关参数（默认就是开启的），用于在 CMS 收集器顶不住要进行 FullGC 时开启内存碎片的合并整理过程，内存整理的过程是无法并发的，空间碎片问题没有了，但停顿时间不得不变长。虚拟机设计者还提供了另外一个参数 -XX：CMSFullGCsBeforeCompaction，这个参数是用于设置执行多少次不压缩的 Full GC 后，跟着来一次带压缩的（默认值为 0，表示每次进入 FullGC 时都进行碎片整理）。


**7. G1 收集器**

G1（Garbage-First），它是一款面向服务端应用的垃圾收集器，在多 CPU 和大内存的场景下有很好的性能。HotSpot 开发团队赋予它的使命是未来可以替换掉 CMS 收集器。

堆被分为新生代和老年代，其它收集器进行收集的范围都是整个新生代或者老年代，而 G1 可以直接对新生代和老年代一起回收。

![img](https://note.youdao.com/yws/api/personal/file/47412B2A05FD4B4097F3AAF66A3A1D9E?method=getImage&version=762&cstk=4sbHpvdq)<br>

G1 把堆划分成多个大小相等的独立区域（Region），新生代和老年代不再物理隔离。

![img](https://note.youdao.com/yws/api/personal/file/8E32A25C50804EA786B07A565AE3D9BD?method=getImage&version=899&cstk=4sbHpvdq)<br>

通过引入 Region 的概念，从而将原来的一整块内存空间划分成多个的小空间，使得每个小空间可以单独进行垃圾回收。这种划分方法带来了很大的灵活性，使得可预测的停顿时间模型成为可能。通过记录每个 Region 垃圾回收时间以及回收所获得的空间（这两个值是通过过去回收的经验获得），并维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region。

G1 收集器遇到的问题：G1 收集器虽然将内存划分为多个独立的 Region；但是假设 Region1 中存在对象 A，且只有 Region2 中的对象 B 引用了对象 A；那么我们确定对象的可达性如果只按照 Region 来看对象 A 是不可达，造成了错误回收。那么要如何判断可达性。难道扫描所有 Region 吗？<br>
解决方案：虚拟机都是使用Remembered Set来避免全堆扫描的；G1中每一reiong都有一个与之对应的remembered set，当程序在写一个Reference类型的变量时，虚拟机会产生一个Write Barrier中断，然后检查要写的变量是否属于同一区域，若不是则用一个CardTable记录下相关的引用信息到被引用对象所属的Region

![img](https://note.youdao.com/yws/api/personal/file/1A3F237F175D40868226A547940CAC17?method=getImage&version=1079&cstk=4sbHpvdq)<br>

如果不计算维护 Remembered Set 的操作，G1 收集器的运作大致可划分为以下几个步骤：

- 初始标记：标记与 GC Root 关联的第一个对象

- 并发标记：从 GC Root 开始找出存活对象

- 最终标记：修正并发标记中表动的那一部分标记，虚拟机将变动的这部分标记记录在 Remembered Set Logs 中，然后合并到 Remembered Set 中。

- 筛选回收：对 Region 回收价值排序，根据用户要求，首先回收价值高的 Region

![img](https://note.youdao.com/yws/api/personal/file/47412B2A05FD4B4097F3AAF66A3A1D9E?method=getImage&version=762&cstk=4sbHpvdq) 

 

## 三、内存分配与回收策略

### 1、内存分配策略

**1. 对象优先在 Eden 分配**

对象有限在新生代的Eden区域分配，当Eden区域的空间不够的时候会触发一次Minor GC。新生代GC（Minor GC）：指发生在新生代的垃圾收集动作，因为Java对象大多都具备朝生夕灭的特性，所以Minor GC非常频繁，一般回收速度也比较快。

老年代GC（Major GC/Full GC）：指发生在老年代的GC，出现了Major GC，经常会伴随至少一次的Minor GC（但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行Major GC的策略选择过程）。Major GC的速度一般会比Minor GC慢10倍以上

```java
/* 优先在新生代的Eden区域分配
 * JVM参数：-Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8 -XX:+PrintGCDetails -XX:+UseSerialGC
 * JAVA对共20M不可扩展；新生代10M；Eden区域8M，Survivor各1M；所以新生代可用为9M 使用Serial收集器
 * @author Bruce
 */
public class EdenFirst {

 	public static void main(String[] args) {

 		System.out.println("Eden First Executing:");

 		//首先在Eden区域分配allocation1、allocation2、allocation3共6M

 		byte[] allocation1 = new byte[2*1024*1024];	//2MB

 		byte[] allocation2 = new byte[2*1024*1024]; //2MB

 		byte[] allocation3 = new byte[2*1024*1024]; //2MB

 		//然后分配4M的allocation4，但是Eden区域的空间已经不够了；触发一次minor GC;

 		//然后我们分析这次GC的过程；使用复制算法；将存活对象allocation1、allocation2、allocation3共6M复制到To Survivor

 		//但是很明显由于To Survivor空间不足以承载Eden+From区域的存活对象；触发空间担保分配；这些存活对象直接进入老年代

 		byte[] allocation4 = new byte[4*1024*1024]; //4MB

 	}

}
```

![img](https://note.youdao.com/yws/api/personal/file/EE160AD6D77449D8B9FF4401281858BD?method=getImage&version=1321&cstk=4sbHpvdq) 


**2. 大对象直接进入老年代**

-XX：PretenureSizeThreshold 参数可以设置一个阈值，超过了这个阈值则视为大对象，会直接分配到老年代。该参数只对 Serial 和 ParNew 两款收集器有效。


**3. 长期存活对象进入老年代**

在新生代中采用复制算法，每经过一次 Minor GC；幸存对象会被移入 Survivor 区域（若 Survivor 区域有足够的空间的话），那么对象每在 Survivor 幸存一次年龄就会加 1；默认情况下年龄大于 15 会移入老年代。参数 -XX：MaxTenuringThreshold 设置晋升老年代的年龄阈值


**4. 动态年龄判断**

在 Survivor 空间中相同年龄所有对象大小的总和大于 Survivor 空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无须等到 MaxTenuringThreshold 中要求的年龄。例如：Survivor 大小为 10M；而又 6  个1M 的同龄对象，那么这些分对象就可以进入老年代。


**5. 空间分配担保**

空间分配担保，在对象优先在 Ede n区域分配的时候我们已经介绍；Eden 区域不够的时候会触发一次 minor GC；但是复制算法时候发现 Eden+From 区域的存活对象大于 To 区域空间，则需要老年代作为担保，包多余的对象直接转到老年代。这就是空间分配担保，但是老年代的空间也不足以容下存活对象怎么办呢？所以空间分配担保分一下三种情况：

- 老年代拥有足够的空间（老年代的剩余空间大于新生代所有存活对象之和），则直接存入老年代

- 老年代空间不足，则检查老年代剩余空间是否大于历次晋升老年代对象均值，若大于且 HandlePromotionFailure 设置为允许担保失败；则尝试 Minor GC

- 老年代空间不足，则检查老年代剩余空间是否大于历次晋升老年代对象均值，若小于或者 HandlePromotionFailure 设置为不允许担保失败；则 Full GC

### 2、 Full GC 的触发条件

对于 Minor GC，其触发条件非常简单，当 Eden 空间满时，就将触发一次 Minor GC。而 Full GC 则相对复杂，有以下条件：

**1. 调用 System.gc()**

只是建议虚拟机执行 Full GC，但是虚拟机不一定真正去执行。不建议使用这种方式，而是让虚拟机管理内存。

**2. 老年代空间不足**

老年代空间不足的常见场景为前文所讲的大对象直接进入老年代、长期存活的对象进入老年代等。

为了避免以上原因引起的 Full GC，应当尽量不要创建过大的对象以及数组。除此之外，可以通过 -Xmn 虚拟机参数调大新生代的大小，让对象尽量在新生代被回收掉，不进入老年代。还可以通过 -XX:MaxTenuringThreshold 调大对象进入老年代的年龄，让对象在新生代多存活一段时间。

**3. 空间分配担保失败**

使用复制算法的 Minor GC 需要老年代的内存空间作担保，如果担保失败会执行一次 Full GC。具体内容请参考上面的第 5 小节。

**4. JDK 1.7 及以前的永久代空间不足**

在 JDK 1.7 及以前，HotSpot 虚拟机中的方法区是用永久代实现的，永久代中存放的为一些 Class 的信息、常量、静态变量等数据。

当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用 CMS GC 的情况下也会执行 Full GC。如果经过 Full GC 仍然回收不了，那么虚拟机会抛出 java.lang.OutOfMemoryError。

为避免以上原因引起的 Full GC，可采用的方法为增大永久代空间或转为使用 CMS GC。

**5. Concurrent Mode Failure**

执行 CMS GC 的过程中同时有对象要放入老年代，而此时老年代空间不足（可能是 GC 过程中浮动垃圾过多导致暂时性的空间不足），便会报 Concurrent Mode Failure 错误，并触发 Full GC。


## 四、虚拟机性能监控工具与故障处理工具

![img](https://note.youdao.com/yws/api/personal/file/47F731765A0D49FBB07021425F62C80E?method=getImage&version=1324&cstk=4sbHpvdq) 

**1. JPS 虚拟机进程状况工具**

![img](https://note.youdao.com/yws/api/personal/file/B2B6998048F24EC7B49DF8EF0A4A6364?method=getImage&version=1323&cstk=4sbHpvdq) 

**2. JSTAT：虚拟机统计信息监视工具**

显示的类装载、内存、垃圾收集、JIT编译等运行数据

![img](https://note.youdao.com/yws/api/personal/file/2AE24DD608174B8C95266C6849A47993?method=getImage&version=1325&cstk=4sbHpvdq) 

**3、Jinfo：配置信息工具**

是实时地查看和调整虚拟机各项参数；可以使用 -flag[+|-]name 或者 -flag name=value 修改一部分运行期可写的虚拟机参数值

jinfo-flag CMSInitiatingOccupancyFraction 1444

-XX：CMSInitiatingOccupancyFraction=85


**4、jmap: Java内存映像工具**

用于生成堆转储快照；以查询 finalize 执行队列、Java 堆和永

久代的详细信息，如空间使用率、当前用的是哪种收集器等

![img](https://note.youdao.com/yws/api/personal/file/5E45C51267004F48A640190B5B26F220?method=getImage&version=1327&cstk=4sbHpvdq) 

 

**5、JSTACK：用于生成虚拟机当前时刻的线程快照**
主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等都是导致线程长时间停顿的常见原因。

![img](https://note.youdao.com/yws/api/personal/file/1555E401803C480EBE6E373071FB5A5A?method=getImage&version=1326&cstk=4sbHpvdq) 

 

## 五、类文件结构

Class 文件是一组以 8 位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在 Class 文件之中，中间没有添加任何分隔符。当遇到需要占用 8 位字节以上空间的数据项时，则会按照高位在前的方式分割成若干个 8 位字节进行存储

|       类型     |         名称        |        备注    |        数量             |
| -------------- | ------------------- | -------------- | ----------------------- |
| u4             | magic               | 魔数           | 1                       |
| u2             | minor_version       | 次版本         | 1                       |
| u2             | major_version       | 主版本         | 1                       |
| u2             | constant_pool_count | 常量池数量     | 1                       |
| cp_info        | constant_pool       | 常量池         | constant_pool_count - 1 |
| u2             | access_flags        | 访问标志       | 1                       |
| u2             | this_class          | 当前类索引     | 1                       |
| u2             | super_class         | 父类索引       | 1                       |
| u2             | interfaces_count    | 接口数量       | 1                       |
| u2             | interfaces          | 接口索引集     | interfaces_count        |
| u2             | fields_count        | 字段数         | 1                       |
| field_info     | fields              | 字段表集       | fields_count            |
| u2             | methods_count       | 方法数         | 1                       |
| method_info    | methods             | 方法表集       | methods_count           |
| u2             | attribute_count     | 属性数         | 1                       |
| attribute_info | attributes          | 属性表集       | attributes_count        |

.class 文件的 0-3 字节表示魔数（为 0xCAFEBABE 标识这是一个 JAVA.class 文件）；后面 4、5 字节标识 Minor Version（次版本号）；6、7 字节标识 Major Version（主版本号：主版本号从 45 开始，每发布一个大版本 +1）



### 1、常量池

用一个 u2 的类型变量（2字节无符号整数）标识常量池有多少个常量；需要说明的是从1开始计数；前面说了 0-7 字节，那么 8、9 字节则标识有多少个常量；例如 0x0013 则标识有 18 个常量（十六进制 19-1；因为从 1 开始）

常量池主要包括字面量以及符号引用：其中字面量就是用 final 关键字声明的变量以及文本字符串；而符号引用包括类和接口的全限定名，字段的名称和描述符，方法的名称和描述符。

| **类型**                    | **标志** | **内容**                                                     | **描述**                     |
| --------------------------- | -------- | ------------------------------------------------------------ | ---------------------------- |
| CONSTANT_Utf8               | 1        | length u2；bytes[length]                                     | UTF-8编码的Unicode字符串     |
| CONSTANT_Integer            | 3        | byte u4                                                      | int类型的字面值              |
| CONSTANT_Float              | 4        | byte u4                                                      | float类型的字面值            |
| CONSTANT_Long               | 5        | byte u8                                                      | long类型的字面值             |
| CONSTANT_Double             | 6        | byteu8                                                       | double类型的字面值           |
| CONSTANT_Class              | 7        | name_index u2 (名字，指向类名utf8)                           | 对一个类或接口的符号引用     |
| CONSTANT_String             | 8        | string_index u2 (指向utf8的索引)                             | String类型字面值的引用       |
| CONSTANT_Fieldref           | 9        | class_index u2 (指向CONSTANT_Class)                          | 对一个字段的符号引用         |
| CONSTANT_Methodref          | 10       | name_and_type_index u2 (指向CONSTANT_NameAndType)            | 对一个类中方法的符号引用     |
| CONSTANT_InterfaceMethodref | 11       | 同上                                                         | 对一个接口中方法的符         |
| CONSTANT_NameAndType        | 12       | name_index u2 (名字，指向utf8) descriptor_index u2 (描述，指向utf8) | 对一个字段或方法的部分符号引 |

通过 constant_pool_counts 得知共有多少常量；诶个常量的结构所占字节数都是固定的；所以我们也就知道对于该类的常量池有多大


### 2、访问标志

常量池结束之后的 16 bits 是访问标志；每一个 bit 代表是否具有该访问修饰符，例如最低位代表 PUBLIC，如果最低位是 1 则标识该类是 PUBLIC 类

| **Flag Name**  | **Value** | **Interpretation**                                  |
| -------------- | --------- | --------------------------------------------------- |
| ACC_PUBLIC     | 0x0001    | public                                              |
| ACC_FINAL      | 0x0010    | final,不能被继承.                                   |
| ACC_SUPER      | 0x0020    | 是否允许使用invokespecial指令，JDK1.2后，该值为true |
| ACC_INTERFACE  | 0x0200    | 是否是接口                                          |
| ACC_ABSTRACT   | 0x0400    | 抽象类                                              |
| ACC_SYNTHETIC  | 0x1000    | 该类不是由用户代码生成,运行时生成的，没有源码       |
| ACC_ANNOTATION | 0x2000    | 是否为注解                                          |
| ACC_ENUM       | 0x4000    | 是否是枚举                                          |


### 3、类/父类/接口索引集

- 类索引：在访问标志之后用一个 u2 类型的值指向常量池中的一个 CONSTANT_Class_info 代表该类的全限定名

- 父类索引：在类索引后用一个 u2 类型的值指向常量池一个 CONSTANT_Class_info 代表父类的全限定名。

- 接口索引集：由于 JAVA 可以实现多个接口，在父类索引后的 u2 类型值标识实现了 interface_counts 个接口；然后依次读取 interface_counts 个 u2 类型值，代表接口全限定名。


### 4、字段表集

接口索引集之后用一个 u2 类型值标识 fields_counts。如何描述一个字段：字段的作用域、是实例变量还是类变量、可变性、并发可见性、可否被序列化、字段数据类型、字段名称

![img](https://note.youdao.com/yws/api/personal/file/B7D36A51776A49739BD6F04EF0B0BD94?method=getImage&version=1301&cstk=4sbHpvdq) 

- access_flags：其中字段的作用域、是否类变量、是否可变、并发可见性、是否可序列化我们可以类似访问标志一样通过u2类型值 access_flags 中的一位代表是否包含该修饰符

![img](https://note.youdao.com/yws/api/personal/file/9A4F4BE172454F0FBCC41F8C6887954B?method=getImage&version=1303&cstk=4sbHpvdq) 

- name_index：指向常量池中的一个 CONSTANT_Utf8_info；该常量由指向了字段的名称

- Descriptor_index：描述符指向常量池中的一个 CONSTANT_Utf8_info；对于数组类型，每一维度将使用一个前置的“[”字符来描述

![img](https://note.youdao.com/yws/api/personal/file/D6170E3D7CCC4499964DFA7BB88FC74E?method=getImage&version=1302&cstk=4sbHpvdq) 

- atrribute_info：属性表集合用于存储一些额外的信息，字段都可以在属性表中描述零至多项的额外信息。


### 5、方法表集

同样首先一个 u2 类型值标识有多少个方法。每个方法表需要描述访问标志（access_flags）、名称索引（name_index）、描述符索引（descriptor_index）、属性表集合（attributes）几

![img](https://note.youdao.com/yws/api/personal/file/EED1F5FD5283490EAC5D8EA5F6F990AA?method=getImage&version=1304&cstk=4sbHpvdq) 


- 访问标志

![img](https://note.youdao.com/yws/api/personal/file/32F49E6053C341389E43C02DFF40F2CC?method=getImage&version=1307&cstk=4sbHpvdq) 

- 名称索引：指向 CONSTANT_Utf8_info

- 描述符：指向 CONSTANT_Utf8_info 标识返回值类型

- 属性表

### 6、属性表集

| **名称**          | **使用者**     | **描述**                   |
| ----------------- | -------------- | -------------------------- |
| Deprecated        | field method   | 字段、方法、类被废弃       |
| ConstantValue     | field          | final常量                  |
| **Code**          | **method**     | **方法的字节码和其他数据** |
| Exceptions        | method         | 方法的异常                 |
| LineNumberTable   | Code_Attribute | 方法行号和字节码映射       |
| LocalVaribleTable | Code_Attribute | 方法局部变量表描述         |
| SourceFile        | Class file     | 源文件名                   |
| Synthetic         | field method   | 编译器产生的方法或字段     |


- Code

![img](https://note.youdao.com/yws/api/personal/file/7CB2616A44A243648A296B32D6AE0AE5?method=getImage&version=1308&cstk=4sbHpvdq) 

max_stack 代表了操作数栈（Operand Stacks）深度的最大值。

max_locals 代表了局部变量表所需的存储空间。

code_length 和 code 用来存储 Java 源程序编译后生成的字节码指令。

异常表的格式如表6-16所示，它包含4个字段，这些字段的含义为：如果当字节码在第

start_pc行[1]到第end_pc行之间（不含第end_pc行）出现了类型为catch_type或者其子类的异常（catch_type为指向一个CONSTANT_Class_info型常量的索引），则转到第handler_pc行继续处理。当catch_type的值为为0时，代表任意异常情况都需要转向到handler_pc处进行处理。

![img](https://note.youdao.com/yws/api/personal/file/0AF76031F6094554A3D37DCFCF537D0E?method=getImage&version=1309&cstk=4sbHpvdq) 

- Exceptions

Exceptions属性的作用是列举出方法中可能抛出的受查异常

![img](https://note.youdao.com/yws/api/personal/file/B5DCEC0BD5BD4D8B965860A2F466002C?method=getImage&version=1310&cstk=4sbHpvdq) 


- LineNumberTable

于描述Java源码行号与字节码行号（字节码的偏移量）之间的对应关系; line_number_info表包括了start_pc和line_number两个u2类型的数据项，前者是字节码行

号，后者是Java源码行号。

![img](https://note.youdao.com/yws/api/personal/file/7D267FF0243D4A3483267B26A5BDA61E?method=getImage&version=1311&cstk=4sbHpvdq) 


- LocalVariableTable

用于描述栈帧中局部变量表中的变量与Java源码中定义的变量之间的关系。

![img](https://note.youdao.com/yws/api/personal/file/FC271E27E53747AB9184D26F54BEC200?method=getImage&version=1313&cstk=4sbHpvdq) 

Start_pc and length描述局部变量作用范围

Name_index and descriptor_index 描述局部变量的名称和类型

index描述在局部变量表中的位置


- ConstantValue

如果同时使用final和static来修饰一个变量（按照习惯，这里称“常量”更贴切），并且这个变量的数据类型是基本类型或者java.lang.String的话，就生成ConstantValue属性来进行初始化，如果这个变量没有被final修饰，或者并非基本类型及字符串，则将会选择在<clinit>方法中进行初始化。

 

## 六、字节码指令

大多数的指令都包含了其操作所对应的数据类型信息，也就是不是每种数据类型和每一种操作都有对应的指令（Boolean、 byte、 char、 short会扩展为整型再计算）

### 1、加载存储指令

- 将一个局部变量加载到操作栈：iload、iload_＜n＞、lload、lload_＜n＞、fload、fload_＜n＞、dload、dload_＜n＞、aload、aload_＜n＞。

- 将一个数值从操作数栈存储到局部变量表：istore、istore_＜n＞、lstore、lstore_＜n＞、fstore、fstore_＜n＞、dstore、dstore_＜n＞、astore、astore_＜n＞。

- 将一个常量加载到操作数栈：bipush、sipush、ldc、ldc_w、ldc2_w、aconst_null、iconst_m1、iconst_＜i＞、lconst_＜l＞、fconst_＜f＞、dconst_＜d＞。

扩充局部变量表的访问索引的指令：wide。


### 2、运算指令

其实Java虚拟机规范没有明确定义过整型数据溢出的具体运算结果，仅规定了在处理整型数据时，只有除法指令（ idiv 和 ldiv）以及求余指令（ irem 和 lrem）中当出现除数为零时会导致虚拟机抛出 ArithmeticException 异常，其余任何整型数运算场景都不应该抛出运行时异常。

![img](https://note.youdao.com/yws/api/personal/file/A73D7F77FD1E47CA9ADD6C216F2FB4FE?method=getImage&version=1312&cstk=4sbHpvdq) 

### 3、类型转换

用于强制类型转换；i2b、i2c、i2s、l2i、f2i、f2l、d2i、d2l和d2f。

### 4、对象创建/访问指令

- 创建类实例的指令：new。

- 创建数组的指令：newarray、anewarray、multianewarray。

- 访问类字段（static字段，或者称为类变量）和实例字段（非static字段，或者称为实例变量）的指令：getfield、putfield、getstatic、putstatic。

- 把一个数组元素加载到操作数栈的指令：baload、caload、saload、iaload、laload、faload、daload、aaload。

- 将一个操作数栈的值存储到数组元素中的指令：bastore、castore、sastore、iastore、fastore、dastore、aastore。

- 取数组长度的指令：arraylength。

- 检查类实例类型的指令：instanceof、checkcast。


### 5、操作数栈管理指令

- 将操作数栈的栈顶一个或两个元素出栈：pop、pop2。

- 复制栈顶一个或两个数值并将复制值或双份的复制值重新压入栈顶：dup、dup2、dup_x1、dup2_x1、dup_x2、dup2_x2。

- 将栈最顶端的两个数值互换：swap。


### 6、控制转移指令

对于long类型、float类型和double类型的条件分支比较操作，则会先执行相应类型的比较运算指令（dcmpg、dcmpl、fcmpg、fcmpl、lcmp，见6.4.3节），运算指令会返回一个整型值到操作数栈中，随后再执行 int 类型的条件分支比较操作来完成整个分支跳转。

- 条件分支：ifeq、iflt、ifle、ifne、ifgt、ifge、ifnull、ifnonnull、if_icmpeq、if_icmpne、if_icmplt、if_icmpgt、if_icmple、if_icmpge、if_acmpeq和if_acmpne。

- 复合条件分支：tableswitch、lookupswitch。

- 无条件分支：goto、goto_w、jsr、jsr_w、ret


### 7、方法调用和返回

- invokevirtual 指令用于调用对象的实例方法，根据对象的实际类型进行分派（虚方法分派），这也是 Java 语言中最常见的方法分派方式。

- invokeinterface 指令用于调用接口方法，它会在运行时搜索一个实现了这个接口方法的对象，找出适合的方法进行调用。

- invokespecial 指令用于调用一些需要特殊处理的实例方法，包括实例初始化方法、私有方法和父类方法。

- invokestatic 指令用于调用类方法（static方法）。

- invokedynamic 指令用于在运行时动态解析出调用点限定符所引用的方法，并执行该方法，前面 4 条调用指令的分派逻辑都固化在 Java 虚拟机内部，而 invokedynamic 指令的分派逻辑是由用户所设定的引导方法决定的。

- ireturn（当返回值是 boolean、byte、char、short 和 int 类型时使用）、lreturn、freturn、dreturn和areturn，另外还有一条 return 指令供声明为 void 的方法、实例初始化方法以及类和接口的类初始化方法使用


### 8、异常处理指令

显式抛出异常的操作（throw 语句）都由 athrow 指令来实现

运行时异常会在其他 Java 虚拟机指令检测到异常状况时自动抛出；如当除数为零时，虚拟机会在 idiv 或 ldiv 指令中抛出 ArithmeticException 异常


### 9、同步指令

同步一段指令集序列通常是由 Java 语言中的 synchronized 语句块来表示的，Java 虚拟机的指令集中有 monitorenter 和monitorexit 两条指令来支持 synchronized 关键字的语义

同步方法使用 Access_flags 实现


## 七、类加载

虚拟机把描述类的数据从 Class 文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的 Java 类型，这就是虚拟机的类加载机制。那什么时候进行类加载呢？

### 1、类加载时机

- 使用 new object() 或访问静态变量静态方法时：遇到 new、getstatic、putstatic 或 invokestatic 这 4 条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。生成这 4 条指令的最常见的 Java 代码场景是：使用 new 关键字实例化对象的时候、读取或设置一个类的静态字段（被 final 修饰、已在编译期把结果放入常量池的静态字段除外）的时候，以及调用一个类的静态方法的时候。

- 使用反射发现未初始化时：使用 java.lang.reflect 包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。

- 父类未加载：当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。

- 虚拟机启动时加载主类：当虚拟机启动时，用户需要指定一个要执行的主类（包含 main() 方法的那个类），虚拟机会先初始化这个主类。

- 当使用 JDK 1.7 的动态语言支持时，如果一个 java.lang.invoke.MethodHandle 实例最后的解析结果 REF_getStatic、REF_putStatic、REF_invokeStatic 的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/335fe19c-4a76-45ab-9320-88c90d6a0d7e.png) 

### 2、加载

加载是类加载的一个阶段，注意不要混淆。

加载过程完成以下三件事：

- 通过类的完全限定名称获取定义该类的二进制字节流。
- 将该字节流表示的静态存储结构转换为方法区的运行时存储结构。
- 在内存中生成一个代表该类的 Class 对象，作为方法区中该类各种数据的访问入口。


其中二进制字节流可以从以下方式中获取：

- 从 ZIP 包读取，成为 JAR、EAR、WAR 格式的基础。
- 从网络中获取，最典型的应用是 Applet。
- 运行时计算生成，例如动态代理技术，在 java.lang.reflect.Proxy 使用 ProxyGenerator.generateProxyClass 的代理类的二进制字节流。
- 由其他文件生成，例如由 JSP 文件生成对应的 Class 类

加载阶段完成后，虚拟机外部的二进制字节流就按照虚拟机所需的格式存储在方法区之中，然后在内存中实例化一个 java.lang.Class 类的对象，这个对象将作为程序访问方法区中的这些类型数据的外部接口。


### 3、验证

这一阶段的目的是为了确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。验证阶段大致上会完成下面 4 个阶段的检验动作：文件格式验证、元数据验证、字节码验证、符号引用验证。

- 文件格式验证：要验证字节流是否符合 Class 文件格式的规范，并且能被当前版本的虚拟机处理。该验证阶段的主要目的是保证输入的字节流能正确地解析并存储于方法区之内，格式上符合描述一个Java类型信息的要求。这阶段的验证是基于二进制字节流进行的，只有通过了这个阶段的验证后，字节流才会进入内存的方法区中进行存储。

- 元数据验证：对字节码描述的信息进行语义分析，以保证其描述的信息符合 Java 语言规范的要求对类的元数据信息进行语义校验，保证不存在不符合 Java 语言规范的元数据信息

- 字节码验证：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。在第二阶段对元数据信息中的数据类型做完校验后，这个阶段将对类的方法体进行校验分析，保证被校验类的方法在运行时不会做出危害虚拟机安全的事件

- 符号引用验证：符号引用验证可以看做是对类自身以外（常量池中的各种符号引用）的信息进行匹配，校验目的是确保解析动作能正常执行。

### 3、准备

是正式为类变量分配内存并设置类变量初始零值的阶段，这些变量所使用的内存都将在方法区中进行分配。（当然如果是静态常量可以通过字段的 Attribute_info 的 CONSTANTValue 获取静态常量值）

### 4、解析

将常量池内的符号引用替换为直接引用的过程，解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符7类符号引用进行

### 5、初始化

通过程序制定的主观计划去初始化类变量和其他资源，也就是执行类构造器<clinit>()方法的过程。

<clinit>()方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块（static{}块）中的语句合并产生的

 

## 八、类加载器

对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立其在Java虚拟机中的唯一性。如果两个类来源于同一个 Class 文件，只要加载它们的类加载器不同，那么这两个类就必定不相等。

### 1、类加载器介绍

从 Java 虚拟机的角度分为两种不同的类加载器：启动类加载器（Bootstrap ClassLoader） 和其他类加载器。其中启动类加载器，使用 C++ 语言实现，是虚拟机自身的一部分；其余的类加载器都由 Java 语言实现，独立于虚拟机之外，并且全都继承自 java.lang.ClassLoader 类。（这里只限于 HotSpot 虚拟机）。

 

### 2、类加载器分类

从 Java 开发人员的角度来看，绝大部分 Java 程序都会使用到以下 3 种系统提供的类加载器。

- 启动类加载器（Bootstrap ClassLoader）

这个类加载器负责将存放在 \lib 目录中的，或者被 -Xbootclasspath 参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如 rt.jar，名字不符合的类库即使放在 lib 目录中也不会被加载）类库加载到虚拟机内存中。

- 扩展类加载器（Extension ClassLoader）

这个加载器由 sun.misc.Launcher$ExtClassLoader 实现，它负责加载 \lib\ext 目录中的，或者被 java.ext.dirs 系统变量所指定的路径中的所有类库，开发者可以直接使用扩展类加载器。

- 应用程序类加载器（Application ClassLoader）

这个类加载器由 sun.misc.Launcher$AppClassLoader 实现。由于这个类加载器是 ClassLoader 中的 getSystemClassLoader() 方法的返回值，所以一般也称它为系统类加载器。它负责加载用户类路径（ClassPath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。


### 3、双亲委派模型


**1. 类加载器分类**

**从 Java 虚拟机的角度来讲，只存在以下两种不同的类加载器：**

- 启动类加载器（Bootstrap ClassLoader），使用 C++ 实现，是虚拟机自身的一部分；

- 所有其它类的加载器，使用 Java 实现，独立于虚拟机，继承自抽象类 java.lang.ClassLoader。

**从 Java 开发人员的角度看，类加载器可以划分得更细致一些：**

- 启动类加载器（Bootstrap ClassLoader）此类加载器负责将存放在 &lt;JRE_HOME\>\lib 目录中的，或者被 -Xbootclasspath 参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如 rt.jar，名字不符合的类库即使放在 lib 目录中也不会被加载）类库加载到虚拟机内存中。启动类加载器无法被 Java 程序直接引用，用户在编写自定义类加载器时，如果需要把加载请求委派给启动类加载器，直接使用 null 代替即可。

- 扩展类加载器（Extension ClassLoader）这个类加载器是由 ExtClassLoader（sun.misc.Launcher$ExtClassLoader）实现的。它负责将 &lt;JAVA_HOME\>/lib/ext 或者被 java.ext.dir 系统变量所指定路径中的所有类库加载到内存中，开发者可以直接使用扩展类加载器。

- 应用程序类加载器（Application ClassLoader）这个类加载器是由 AppClassLoader（sun.misc.Launcher$AppClassLoader）实现的。由于这个类加载器是 ClassLoader 中的 getSystemClassLoader() 方法的返回值，因此一般称为系统类加载器。它负责加载用户类路径（ClassPath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。


**2. 工作过程**

应用程序是由三种类加载器互相配合从而实现类加载，除此之外还可以加入自己定义的类加载器。

下图展示了类加载器之间的层次关系，称为双亲委派模型（Parents Delegation Model）。该模型要求除了顶层的启动类加载器外，其它的类加载器都要有自己的父类加载器。这里的父子关系一般通过组合关系（Composition）来实现，而不是继承关系（Inheritance）。

![img](https://note.youdao.com/yws/api/personal/file/B8B506D273CE4E2AAF3F34037E636267?method=getImage&version=629&cstk=4sbHpvdq)<br>

一个类加载器首先将类加载请求转发到父类加载器，只有当父类加载器无法完成时才尝试自己加载。

**3. 好处**

使得 Java 类随着它的类加载器一起具有一种带有优先级的层次关系，从而使得基础类得到统一。

例如 java.lang.Object 存放在 rt.jar 中，如果编写另外一个 java.lang.Object 并放到 ClassPath 中，程序可以编译通过。由于双亲委派模型的存在，所以在 rt.jar 中的 Object 比在 ClassPath 中的 Object 优先级更高，这是因为 rt.jar 中的 Object 使用的是启动类加载器，而 ClassPath 中的 Object 使用的是应用程序类加载器。rt.jar 中的 Object 优先级更高，那么程序中所有的 Object 都是这个 Object。

**4. 实现**

以下是抽象类 java.lang.ClassLoader 的代码片段，其中的 loadClass() 方法运行过程如下：先检查类是否已经加载过，如果没有则让父类加载器去加载。当父类加载器加载失败时抛出 ClassNotFoundException，此时尝试自己去加载。

```java
public abstract class ClassLoader {
    // The parent class loader for delegation
    private final ClassLoader parent;

    public Class<?> loadClass(String name) throws ClassNotFoundException {
        return loadClass(name, false);
    }

    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        throw new ClassNotFoundException(name);
    }
}
```

### 4、破坏双亲委派模型

双亲委派模型主要出现过3次较大规模“被破坏”的情况。

- 第一次破坏是因为类加载器和抽象类 java.lang.ClassLoader 在 JDK1.0 就存在的，而双亲委派模型在 JDK1.2 之后才被引入，为了兼容已经存在的用户自定义类加载器，引入双亲委派模型时做了一定的妥协：在 java.lang.ClassLoader 中引入了一个 findClass() 方法，在此之前，用户去继承 java.lang.Classloader 的唯一目的就是重写 loadClass() 方法。JDK1.2 之后不提倡用户去覆盖 loadClass() 方法，而是把自己的类加载逻辑写到 findClass() 方法中，如果 loadClass() 方法中如果父类加载失败，则会调用自己的 findClass() 方法来完成加载，这样就可以保证新写出来的类加载器是符合双亲委派模型规则的。

- 第二次破坏是因为模型自身的缺陷，现实中存在这样的场景：基础的类加载器需要求调用用户的代码，而基础的类加载器可能不认识用户的代码。为此，Java设计团队引入的设计时“线程上下文类加载器（Thread Context ClassLoader）”。这样可以通过父类加载器请求子类加载器去完成类加载动作。已经违背了双亲委派模型的一般性原则。

- 第三次破坏 是由于用户对程序动态性的追求导致的。这里所说的动态性是指：“代码热替换”、“模块热部署”等等比较热门的词。说白了就是希望应用程序能够像我们的计算机外设一样，接上鼠标、U盘不用重启机器就能立即使用。OSGi是当前业界“事实上”的Java模块化标准，OSGi实现模块化热部署的关键是它自定义的类加载器机制的实现。每一个程序模块（OSGi中称为Bundle）都有一个自己的类加载器，当需要更换一个Bundle时，就把Bundle连同类加载器一起换掉以实现代码的热替换。在OSGi环境下，类加载器不再是双亲委派模型中的树状结构，而是进一步发展为更加复杂的网状结构。

### 5、自定义类加载器实现

以下代码中的 FileSystemClassLoader 是自定义类加载器，继承自 java.lang.ClassLoader，用于加载文件系统上的类。它首先根据类的全名在文件系统上查找类的字节代码文件（.class 文件），然后读取该文件内容，最后通过 defineClass() 方法来把这些字节代码转换成 java.lang.Class 类的实例。

java.lang.ClassLoader 的 loadClass() 实现了双亲委派模型的逻辑，自定义类加载器一般不去重写它，但是需要重写 findClass() 方法。

```java
public class FileSystemClassLoader extends ClassLoader {

    private String rootDir;

    public FileSystemClassLoader(String rootDir) {
        this.rootDir = rootDir;
    }

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = getClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException();
        } else {
            return defineClass(name, classData, 0, classData.length);
        }
    }

    private byte[] getClassData(String className) {
        String path = classNameToPath(className);
        try {
            InputStream ins = new FileInputStream(path);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 4096;
            byte[] buffer = new byte[bufferSize];
            int bytesNumRead;
            while ((bytesNumRead = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, bytesNumRead);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    private String classNameToPath(String className) {
        return rootDir + File.separatorChar
                + className.replace('.', File.separatorChar) + ".class";
    }
}
```
