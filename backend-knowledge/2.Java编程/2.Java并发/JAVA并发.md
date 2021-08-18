# JAVA并发编程

[toc]

## 一、线程基础

#### 1、程序、进程和线程

程序是代码和数据存储在磁盘上的静态文件；

进程是程序在某数据集上的一次执行；

线程是轻量级的进程（对于如何理解，用了就知道了. ；


#### 2、为什么需要线程

- 使用线程能够提高资源利用率及效率：假设程序只有一个线程（主线程main. ，若执行到System.in则阻塞，此时该进程只能够等待用户输入不能执行其它操作。而如果是多线程等待用户输入的过程中可以执行其它操作。

- 那么多进程也可以提高整体系统资源利用率：当某个进程进入等待时，其它线程可以获得CPU执行；资源利用率也提高了为什么还需要线程？

    - 对于当前线程来说，使用多线程，那么当前线程的利用率高。
    
    - 进程的切换代价远比线程切换代价要高，使用多线程可以获得更好的并发度。

- 业务模型需要：现实世界中往往很多事物是并发（并行）的，比如我可以一边听歌一边学习，所以并发就是对事物描述自然而然的结果。


#### 3、线程的基本使用

- 实现Runnable接口，并实现run方法

- 继承Thread类（Thread类本身也实现Runable接口） ，并重载 run 方法（或传入一个可运行对象) 

注意：最好实现Runable接口创建线程类，然后实例化该线程类传入Thread创建线程；因为JAVA是单继承的继承了Thread类就不能继承其他类的，但是实现Runnable还可以继承其它类，实现其他接口。  


#### 4、Thread类中的基本方法

- 线程状态相关：start(); sleep(); join();yeild();

- 其他基本方法：setName(); currentThread(); getName(); setPriority();

- 弃用方法：stop(); suspend(); resume() 等不做过多说明

线程基础源码示例1：base.BaseStudy


#### 5、线程状态总结

一个线程只能处于一种状态，并且这里的线程状态特指 Java 虚拟机的线程状态，不能反映线程在特定操作系统下的状态。

**1. 新建（NEW）**

创建后尚未启动。

**2. 可运行（RUNABLE）**

正在 Java 虚拟机中运行。但是在操作系统层面，它可能处于运行状态，也可能等待资源调度（例如处理器资源），资源调度完成就进入运行状态。所以该状态的可运行是指可以被运行，具体有没有运行要看底层操作系统的资源调度。

**3. 阻塞（BLOCKED）**

请求获取 monitor lock 从而进入 synchronized 函数或者代码块，但是其它线程已经占用了该 monitor lock，所以出于阻塞状态。要结束该状态进入从而 Runable 需要其他线程释放 monitor lock。

**4. 无限期等待（WAITING）**

等待其它线程显式地唤醒。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取 monitor lock。而等待是主动的，通过调用  Object.wait() 等方法进入。

| 进入方法 | 退出方法 |
| --- | --- |
| 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
| 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕 |
| LockSupport.park() 方法 | LockSupport.unpark(Thread) |

**5. 限期等待（TIMED_WAITING）**

无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

| 进入方法 | 退出方法 |
| --- | --- |
| Thread.sleep() 方法 | 时间结束 |
| 设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll()  |
| 设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕 |
| LockSupport.parkNanos() 方法 | LockSupport.unpark(Thread) |
| LockSupport.parkUntil() 方法 | LockSupport.unpark(Thread) |

调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述。睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。

**6. 结束（TERMINATED）**

可以是线程结束任务之后自己结束，或者产生了异常而结束。

注意：sleep()和wait()的区别：sleep(不会失去任何线程的monitor，而wait()会释放锁，并且wait()需要在synchronized代码块中使用，sleep()是Thread的方法，而wait()是每个对象的方法。


#### 6、线程中断

1. Thread.interrupt() 设置中断标志位true

2. Thread.interrupted() 获取中断标志位,且清空中断标志位；

3. Thread.isInterrupted() 获取中断标志位，不清空中断标志位；

线程基础源码示例2：base.ThreadInterrput

 

## 二、JAVA内存模型

#### 1、JAVA内存模型简述

- 方法区：常量池、类信息、静态变量、方法的字节码等

- 堆：用于存放对象实例

- 程序计数器：用于存放下一条指令地址

- 虚拟机栈：由栈帧构成；每一个方法执行一次就会创建一个栈帧，执行完成自动销毁；每一个栈帧包括（局部变量、操作数栈、返回地址等. 

- 本地方法栈：类似虚拟机栈，供 native 执行

    ![img](https://note.youdao.com/yws/api/personal/file/A87619A8B08B4F31960B0D6F0BC43CE2?method=getImage&version=1335&cstk=6lqG8sux) 

JVM定义了一组规则，通过这组规则来决定一个线程对共享变量的写入何时对另一个线程可见，这组规则也称为Java内存模型（即JMM) ，JMM是围绕着程序执行的原子性、有序性、可见性展开的。

#### 2、主内存与工作内存

处理器上的寄存器的读写的速度比内存快几个数量级，为了解决这种速度矛盾，在它们之间加入了高速缓存。

加入高速缓存带来了一个新的问题：缓存一致性。如果多个缓存共享同一块主内存区域，那么多个缓存的数据可能会不一致，需要一些协议来解决这个问题。

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/942ca0d2-9d5c-45a4-89cb-5fd89b61913f.png)<br>

所有的变量都存储在主内存中，每个线程还有自己的工作内存，工作内存存储在高速缓存或者寄存器中，保存了该线程使用的变量的主内存副本拷贝。

线程只能直接操作工作内存中的变量，不同线程之间的变量值传递需要通过主内存来完成。

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/15851555-5abc-497d-ad34-efed10f43a6b.png)<br>

#### 3、内存间交互操作

Java 内存模型定义了 8 个操作来完成主内存和工作内存的交互操作。

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/8b7ebbad-9604-4375-84e3-f412099d170c.png)<br>

- read：把一个变量的值从主内存传输到工作内存中
- load：在 read 之后执行，把 read 得到的值放入工作内存的变量副本中
- use：把工作内存中一个变量的值传递给执行引擎
- assign：把一个从执行引擎接收到的值赋给工作内存的变量
- store：把工作内存的一个变量的值传送到主内存中
- write：在 store 之后执行，把 store 得到的值放入主内存的变量中
- lock：作用于主内存的变量
- unlock


#### 4、原子性

这里原子性指的是一个原子操作，不可被中断；而非加锁保证的代码块“原子性”。粗浅的理解为一条JAVA字节码指令就是一个原子操作。Java 内存模型保证了 read、load、use、assign、store、write、lock 和 unlock 操作具有原子性，例如对一个 int 类型的变量执行 assign 赋值操作，这个操作就是原子性的。但是 Java 内存模型允许虚拟机将没有被 volatile 修饰的 64 位数据（long，double）的读写操作划分为两次 32 位的操作来进行，即 load、store、read 和 write 操作可以不具备原子性。

例如：i++；其实是多条指令，所以i++不是原子操作

1.先加载 i; 2.对 i 执行自加; 3.回写自加后的 i;

#### 5、指令重排

为了提高性能，编译器和处理器的常常会对指令做重排

每条指令执行分为不同的步骤，需要用到不同的硬件设备。我们可以简单的理解为：<br>
取指令地址：IF；译码和取存储器操作数：ID；执行或有效地址计算：EX；存储器访问：MEM；写回：WB；

指令按照流水线执行会存在“空节拍”，空节拍会大大降低执行效率，因此希望能够进行指令重排以去掉空节拍，提高效率。

指令重排分类：编译器优化重排、指令并行重排（第二点所描述）、内存系统重排

注意：指令重排仅保证串行语义，也就是保证单线程语句之间的依赖关系。


#### 6、可见性

当前线程对于共享变量的修改对于其他线程来说能够立即可见。需要说明的是能够立即可见，并不代表一定立即可见，其他线程需要读取修改后的值才能够知道已经修改，若不读取自然不知道。

对于单线程而言，一定满足可见性；但是对于多线程而言，每个线程有自己的程序计数器，虚拟机栈，本地方法栈；而共享变量通常存在堆中（常量池不可变一定可见） ，某个线程修改共享变量后何时回写，其他线程何时读取修改后的值都是不确定的，因此造成了多线程的不可见。


#### 7、有序性

尽管存在指令重排；但是仍然保证单线程语义上的顺序（在实际执行过程仍然是无序的. ，而对于多线程而言，指令重排就不能保证多线程编发执行下的有序性了。

所以我们需要一系列的手段保证多线程下的原子性、可见性、有序性


#### 8、先行发生原则

所有的指令重排都应该建立在现行发生原则的基础之上：如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。

1. 单一线程原则：单个线程内保证串行语义

2. 管程锁定规则：unlock 必然发生在后续同一锁加锁之前

3. volatile 变量规则：volatile 变量写优先发生于读，即每次读 volatile 变量都从主存读，每次更新立即回写到主存。

4. 线程启动规则：线程的 start() 优先与其它操作

5. 线程加入规则：线程的所有操作做优先于终止操作

6. 线程中断规则：线程 interrupt() 方法的调用先行于被中断线程的代码检测到中断事件的发生

7. 传递性：A 先于 B ，B 先于 C  那么 A 必然先于 C

8. 对象终结规则：对象的构造函数执行，结束先于 finalize() 方法


#### 9、线程实现

Java线程的实现是基于一对一的线程模型，通过语言级别层面程序去间接调用系统内核的线程模型。即我们在使用Java线程时，Java虚拟机内部是转而调用当前操作系统的内核线程来完成当前任务。内核线程(Kernel-Level Thread，KLT)，它是由操作系统内核(Kernel)支持的线程，这种线程是由操作系统内核来完成线程切换，内核通过操作调度器进而对线程执行调度，并将线程的任务映射到各个处理器上。

 

## 三、线程同步基础

#### 1、基本概念

线程安全问题：当多个线程同时读写共享资源（类变量） 时，可能造成结果错误

线程安全：多线程并发跟多个线程串行执行结果一致

线程不安全：多个线程并发执行结果可能和多个线程串行执行结果不一致

如何解决线程不安全问题：阻塞（加锁） 、无阻塞（无锁）

线程安全示例源码3：threadsecurity.ThreadSecurity

#### 2、synchronized关键字使用

synchronized 是从 JVM 层面解决线程安全问题。 JAVA中 每个对象都有一个监视器，来监测并发代码的重入；当线程要进入synchronized代码执行时需要先获取 synchronized 作用对象的监视器；若取得监视器则进入执行，执行完之后释放监视器；若监视器已被其他线程占有，则阻塞在 synchronized 作用对象上

1. synchronized 代码块：锁定执行对象（只能是引用类型，不可是基本类型）

2. Synchronized 修饰实例方法：锁定this

3. synchronized 修饰类方法：锁定该类所有实例（锁定的是当前类的 Class 对象） 

注意1：synchronized 使用错误：当使用 synchronized 方法修饰实例方法时，锁定了不同实例无法解决线程不安全问题。

注意2：synchronized 锁定 String、Integer 等不可变对象是线程不安全的，假设在线程run方法里面修改了String、Integer对象实际上是重新创建了一个新的对象，那么多个线程就锁定了不同的对象从而线程不安全。

注意2：Synchronized 是可重入的，不可重入可能会导致死锁。

注意3：Synchronized 阻塞不响应中断

线程安全示例源码4：threadsecurity.ThreadSecuritySynchronized


#### 3、线程通信

1. wait()/notify() 是对象的方法，而不是线程的方法

2. wait()/notify() 必须在 synchronized 代码块中，否则会报错

3. wait()/notify() 必须是 synchronized 的同一个对象，否则会报错

线程通信示例源码5：threadsecurity.Synchronized_Wait

 

#### 4、synchronized关键字实现

**1. 对象在内存中的存储**

- 实例变量：存放类的属性数据信息，包括父类的属性信息，如果是数组的实例部分还包括数组的长度。

- 填充数据：虚拟机要求对象起始地址必须是8字节的整数倍。填充数据不是必须存在的，仅仅是为了字节对齐。

- 对象头结构及 synchronized 锁类型

![img](https://note.youdao.com/yws/api/personal/file/EF3A15A2CBCB430D81B5ECEFF8E0C5DC?method=getImage&version=1291&cstk=6lqG8sux) 

| 锁状态   | 25bit          | 4bit     | 1bit是否偏向 | 2bit锁标记 |
| -------- | -------------- | -------- | ------------ | ---------- |
| 无锁     | 对象hashcode   | 分代年龄 | 0            | 01         |
| 偏向锁   | Thread ID      | Epoch    | 1            | 01         |
| 轻量级锁 | 指向栈中锁记录 |          |              | 00         |
| 重量级锁 | 指向monitor    |          |              | 10         |
| GC标记   | 空             |          |              | 11         |


上表所示32位虚拟机中对象头的结构，对象头一般为2个字（当对象为数组的时会是3个字，用一个字存储数组长度. 。当锁标志位是‘10’时代表是重量锁，对象头中保存对象关联的 monitor 实例引用，以下是 monitor 的实现中主要字段：

```c
ObjectMonitor() {

    _count = 0; 

    _owner = NULL;

    _WaitSet = NULL;

    _EntryList = NULL;

    …………………………
}
```

无论是基于代码块还是方法，synchronized 通过与锁定对象关联的 monitor 实现重量锁。监视器有   \_WaitSet 和  \_EntryList 队列，以及  \_Owner 标记： 

\_WaitSet 是调用 wait() 方法的等待队列；<br>
\_EntryList 是要进入同步代码块的线程阻塞队列；<br>
\_Owner 是正在同步代码块执行的线程标记。当有线程要进入同步代码块首先到达 \_EntryList 阻塞；判断 \_Owner 标记是否为 null；若为 null 则 _EntryList; 

队列中的线程可竞争同步对象监视器；获取监视器的线程将 _Owner 标记设为自身，并将 monitor 中的count+1；若同步代码块中的线程执行完毕或者调用 wait() 进入 _WaitSet 队列中，则释放 monitor;  _Owner 置位 Null; count-1

**2. synchronized 同步语句块的实现**

使用的是 monitorenter 和 monitorexit 指令，其中monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置，当执行 monitorenter 指令时，当前线程将试图获取 objectref (被锁定对象) 所对应的 monitor 的持有权，当 objectref 的 monitor 的进入计数器为 0，那线程可以成功取得 monitor，并将计数器值设置为 1，取锁成功。如果当前线程已经拥有 objectref 的 monitor 的持有权，那它可以重入这个 monitor， 重入时计数器的值也会加 1。倘若其他线程已经拥有 objectref  的 monitor 的所有权，那当前线程将被阻塞，直到正在执行线程执行完毕，即 monitorexit 指令被执行，执行线程将释放  monitor(锁)并设置计数器值为0 ，其他线程将有机会持有 monitor 。

需要注意的是编译器将会确保无论方法通过何种方式完成，方法中调用过的每条 monitorenter 指令都有执行其对应 monitorexit 指令，而无论这个方法是正常结束还是异常结束。为了保证在方法异常完成时 monitorenter 和 monitorexit 指令依然可以正确配对执行，编译器会自动产生一个异常处理器，这个异常处理器声明可处理所有的异常，它的目的就是用来执行 monitorexit 指令。从字节码中也可以看出多了一个monitorexit指令，它就是异常结束时被执行的释放monitor 的指令。

**3. 方法级synchronized实现**

方法级的同步是隐式，即无需通过字节码指令来控制的，它实现在方法调用和返回操作之中。JVM可以从方法常量池中的方法表结构(method\_info Structure) 中的 ACC\_SYNCHRONIZED 访问标志区分一个方法是否同步方法。当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先持有monitor， 然后再执行方法，最后在方法完成(无论是正常完成还是非正常完成)时释放 monitor。 在方法执行期间，执行线程持有了monitor，其他任何线程都无法再获得同一个 monitor。 如果一个同步方法执行期间抛 出了异常，并且在方法内部无法处理此异常，那这个同步方法所持有的 monitor 将在异常抛到同步方法之外时自动释放

在Java早期版本中，synchronized属于重量级锁，效率低下，因为监视器锁 monitor 是依赖于底层的操作系统的 Mutex Lock 来实现的，而操作系统实现线程之间的切换时需要从用户态转换到核心态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高。

 

#### 5、synchronized改进

先提供偏向锁，如果不满足的时候，升级为轻量级锁，再不满足，升级为重量级锁。自旋锁是一个过渡的锁状态，不是一种实际的锁类型。

**1. 偏向锁**

在大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，因此为了减少同一线程获取锁的代价而引入偏向锁。偏向锁的思想是偏向于让第一个获取锁对象的线程，这个线程在之后获取该锁就不再需要进行同步操作，甚至连 CAS 操作也不再需要。

当锁对象第一次被线程获得的时候，进入偏向状态，标记为 1 01。同时使用 CAS 操作将线程 ID 记录到 Mark Word 中，如果 CAS 操作成功，这个线程以后每次进入这个锁相关的同步块就不需要再进行任何同步操作。

当有另外一个线程去尝试获取这个锁对象时，偏向状态就宣告结束，此时撤销偏向（Revoke Bias）后恢复到未锁定状态或者轻量级锁状态。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/390c913b-5f31-444f-bbdb-2b88b688e7ce.jpg" width="600"/> </div><br>

2. 轻量级锁

倘若偏向锁失败，虚拟机并不会立即升级为重量级锁，它还会尝试使用一种称为轻量级锁的优化手段，此时Mark Word 的结构也变为轻量级锁的结构。轻量级锁能够提升程序性能的依据是“对绝大部分的锁，在整个同步周期内都不存在竞争”，注意这是经验数据。需要了解的是，轻量级锁所适应的场景是线程交替执行同步块的场合，如果存在同一时间访问同一锁的场合，就会导致轻量级锁膨胀为重量级锁及重量锁实现. 

3. 自旋锁

轻量级锁失败后，虚拟机为了避免线程真实地在操作系统层面挂起，还会进行一项称为自旋锁的优化手段。这是基于在大多数情况下，线程持有锁的时间都不会太长，如果直接挂起操作系统层面的线程可能会得不偿失，毕竟操作系统实现线程之间的切换时需要从用户态转换到核心态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，因此自旋锁会假设在不久将来，当前的线程可以获得锁，因此虚拟机会让当前想要获取锁的线程做几个空循环(这也是称为自旋的原因)，一般不会太久，可能是50个循环或100循环，在经过若干次循环后，如果得到锁，就顺利进入临界区。如果还不能获得锁，那就会将线程在操作系统层面挂起，这就是自旋锁的优化方式，这种方式确实也是可以提升效率的。最后没办法也就只能升级为重量级锁了。

在 JDK 1.6 中引入了自适应的自旋锁。自适应意味着自旋的次数不再固定了，而是由前一次在同一个锁上的自旋次数及锁的拥有者的状态来决定。

4. 锁消除

消除锁是虚拟机另外一种锁的优化，这种优化更彻底，Java虚拟机在JIT编译时(可以简单理解为当某段代码即将第一次被执行时进行编译，又称即时编译)，通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁，通过这种方式消除没有必要的锁，可以节省毫无意义的请求锁时间，如下 StringBuffe r的 append 是一个同步方法，但是在 add 方法中的 StringBuffer 属于一个局部变量，并且不会被其他线程所使用，因此 StringBuffer 不可能存在共享资源竞争的情景， JVM 会自动将其锁消除。

 

#### 6、volatile

1. 语义：当写一个 volatile 变量时，JMM 会把该线程对应的工作内存中的共享变量值刷新到主内存中；当读取一个 volatile 变量时，JMM 会把该线程对应的工作内存置为无效，那么该线程将只能从主内存中重新读取共享变量。禁止指令重排（内存屏障）。

2. 既然 volatile 保证了可见性，那么为什么还是不能保证线程安全呢？

举例说明：两个线程 Thread1 and Thread2 对静态变量 static volatile int i = 0; 执行i++；若Thread1 and Thread2同时读取 i = 0；然后Thread1执行i++，并立即回写到堆中；而此时其实不能保证可见性，因为 Thread2 已经读取了 i 的值，已经认定 i 就是等于0了，结果是 Thread2 执行 i++ 将 i=1 回写到堆，输出仍然是1，线程不安全。

3. 那么 volatile 有什么用？要是 Thread1 执行完 i++ 回写的时候，能后直接告诉 Thread2, 它已经读取的 i 是失效的就应该可以解决线程安全问题。那么用不用 volatile 的区别到底在哪里？还是上面的例子：若 Thread1 执行i++; Thread2 执行 i++; i++；也是 Tread1 and Thread2 都读取i = 0；

- 不使用 volatile：Thread1 执行完 i++；但是可能没有立即回写 i = 1；然后 Thread2 执行完 i++; i=1 但是它没有回写，接着执行 i++;（也没有到堆中重新读取 i 的值） ；i++ 回写 i = 2;  Thread1 又来回写 i = 1;

- 使用volatile：Thread1先执行i++；立即回写i = 1；Thread2 执行 i++ 回写 i = 1；然后读取对中内存 i = 1 并执行 i++; 回写 i = 2；

3. 禁止指令重排：通过内存屏障实现（又称内存栅栏），是一个 CPU 指令，它的作用有两个，一是保证特定操作的执行顺序，二是保证某些变量的内存可见性。如果在指令间插入一条 Memory Barrier 则会告诉编译器和 CPU，不管什么指令都不能和这条 Memory Barrier 指令重排序，也就是说通过插入内存屏障禁止在内存屏障前后的指令执行重排序优化。Memory Barrier 的另外一个作用是强制刷出各种 CPU 的缓存数据，因此任何 CPU 上的线程都能读取到这些数据的最新版本。

 

## 四、线程同步进阶

#### 1、AtomicInteger：原子整数

1. private volatile value;	value是AtomicInteger封装的真正要需操作的整型值

2. valueOffset表示value字段在AtomicInteger这个类的相对偏移量
```java
static {

   try {

       valueOffset = unsafe.objectFieldOffset

       (AtomicInteger.class.getDeclaredField("value"));

   } catch (Exception ex) { throw new Error(ex); }

  }
```
3. CAS操作： compareAndSwapInt 取当前对象 this 的 value 字段（通过 valueOffset 偏移量获取）要跟新 value 需要提供一个期望值 expect，若期望值与 value 真是值相同则将 update 赋值给value; CAS 是原子操作。
```java
public final boolean compareAndSet(int expect, int update) {

     return  unsafe.compareAndSwapInt(this, valueOffset, expect, 	update);

  }
```
4. 如何用 CAS 实现原子操作：通过 get() 获取 value 当前值；对 current + 1; 若此时有其他线程改变value值，则CAS不通过进入下一轮循环。
```java
public final int getAndIncrement() {

     for (;;) {

       int current = get();

       int next = current + 1;

       if (compareAndSet(current, next))

         return current;

     }

}
```


#### 2、AtomicReference：支持泛型

1. 封装的引用 value

```private volatile V value;```

2. valueOffset表示value字段在AtomicReference这个类的相对偏移量
```java
static {

   try {

     valueOffset = unsafe.objectFieldOffset

       (AtomicReference.class.getDeclaredField("value"));

   } catch (Exception ex) { throw new Error(ex); }

  }
```
3. CAS操作：compareAndSwapInt取当前对象 this 的 value 字段（通过 valueOffset 偏移量获取. 要跟新 value 需要提供一个期望值expect，若期望值与 value 真是值相同则将 update 赋值给 value; CAS 是原子操作。
```java
public final boolean compareAndSet(V expect, V update) {

     return unsafe.compareAndSwapObject(this, valueOffset, expect, update);

  }
```
4. 更新value值之前；先获取当前value；然后利用CAS操作更新value；若此时有其他线程修改了value值，则此轮修改不成功。
```java
public final V getAndSet(V newValue) {

     while (true) {

       V x = get();

       if (compareAndSet(x, newValue))

         return x;

     }

  }
```
#### 3、AtomicIntegerArray：原子整数数组

1. 真正封装的int[] array

```private final int[] array;```

2. 获取base、shift、sacle
```java
private static final int base = unsafe.arrayBaseOffset(int[].class);   //获取第一个元素偏移量

static { //scale是数组元素大小,eg: int = 4

     int scale = unsafe.arrayIndexScale(int[].class);

     if ((scale & (scale - 1)) != 0)

       throw new Error("data type scale not a power of two");

       shift = 31 - Integer.numberOfLeadingZeros(scale);

  } scale = 4;前导零 = 29;所以shift = 2
```
3. 定位元素位置（相对偏移量）:第 i 个元素，左移4位及找到第i个元素偏移量；例如第一个元素相对偏移量是8；第i = 2个元素相对偏移量应该是i * 4 + 8 = 16;
```java
private static long byteOffset(int i) {

     return ((long) i << shift) + base;

  }
```
4. 使用checkedByteOffset获取第 i 个元素的偏移量，然后使用CAS操作设置第i个元素的值。
```java
public final int getAndSet(int i, int newValue) {

     long offset = checkedByteOffset(i);

     while (true) {

       int current = getRaw(offset);

       if (compareAndSetRaw(offset, current, newValue))

         return current;

     }

  }

private boolean compareAndSetRaw(long offset, int expect, int update) {

     return *\unsafe\\*.compareAndSwapInt(array, offset, expect, update);

}
```


#### 4、AtomicIntegerFieldUpdater

使得对一个volatile int变量的操作具备原子性。

应用场景：由于项目前期考虑不周全，项目需求又发生变化，使得某个类中的变量需要执行多线程操作，由于该变量多处使用，改动起来比较麻烦，而且原来使用的地方无需使用线程安全，只要求新场景需要使用时。

原子更新器的使用存在比较苛刻的条件如下：

1. 操作的字段不能是static类型。

2. 操作的字段不能是final类型的，因为final根本没法修改。

3. 字段必须是volatile修饰的，也就是数据本身是读一致的。

4. 属性必须对当前的Updater所在的区域是可见的

当构建一个Updater的时候需要提供相应类的Class对象，以及需要更新的字段名，在构造函数中会通过反射获取该字段的Field对象，然后在获取偏移量；后面的内容和其他的都一样了。

 

#### 5、AtomicStampedReference

1. ABA问题：若有共享变量 AtomicInteger i = new AtomicInteger(10)，Thread1将 i + 10，但是只能增加一次；Thread2将i - 10；Thread1当i=10则增加10；Thread2当i => 10,就减10；Thread1对i增加10；然后Thread2又减掉10,此时Thread1又发现i = 10，又执行加10操作。这显然不能满足我们的需求，向这样与过程相关的操作，我们需要使用Stamped（可以理解为时戳） 

2. AtomicStampedReference将需要操作的reference和一个Stamp封装成一个Pair
```java
private volatile Pair<V> pair;

private static class Pair<T> {

     final T reference;

     final int stamp;

     private Pair(T reference, int stamp) {

       this.reference = reference;

       this.stamp = stamp;

     }

     static <T> Pair<T> of(T reference, int stamp) {

       return new Pair<T>(reference, stamp);

     }

  }
```
3. 对于AtomicIntegerFieldUpdater的修改不仅需要提供期望值，还需要提供期望的stamp
```java
public boolean compareAndSet(V  expectedReference,

                  V  newReference,

                  int expectedStamp,

                  int newStamp) {

     Pair<V> current = pair;

     return

       expectedReference == current.reference &&

       expectedStamp == current.stamp &&

       ((newReference == current.reference &&

        newStamp == current.stamp) ||

        casPair(current, Pair.of(newReference, newStamp)));

  }
```


#### 6、AbstractQueuedSynchronizer

AQS：抽象队列同步器，是用于构建其他同步组件的基础框架。其中包含整型的 state 属性，用于表示共享资源的可用状态。如果被请求的共享资源处于可用状态则修改 state 并将当前线程设置为有效工作线程，如共享资源处于不可用该状态则通过一套机制处理线程的阻塞与唤醒，在 AQS 中是通过 CLH 队列实现。

CLH 是一个双向队列，每一个节点都是一个 Node 对象，Node 是 AQS 的内部类，用于封装线程。AQS 中会持有 CLH 的头部 head 和尾部 tail；Node 主要的属性包括被封装的线程、waitStatus、next、prev、nextWaiter；其中 waitStatus 取值 CANCELLED、CONDITION、SIGNAL、PROPAGATE。节点的共享方式，独占式和共享式。当线程请求共享资源处于不可用状态时被封装成Node加入到同步队列中，如果调用 await() 方法则加入等待队列中。

AQS 是基于模板方法模式的，不同的同步器只主要去实现获取锁和释放锁的逻辑即可；独占式的实现 tryAcquire(); tryRelease()；共享式实现 tryAcquireShared(); tryReleaseShared();

```java
public abstract class AbstractQueuedSynchronizer

extends AbstractOwnableSynchronizer{

//指向同步队列队头

private transient volatile Node head;

//指向同步的队尾

private transient volatile Node tail;

//同步状态，0代表锁未被占用，大于0代表锁已被占用

private volatile int state;

//内部类 Node

static final class Node {

volatile int waitStatus;

volatile Node prev;

volatile Node next;

volatile Thread thread;

Node nextWaiter;

}

//省略其他代码......

}
```
#### 7、ReentrantLock

1. 重入锁使用：示例源码6：threadsecurity.ReentrantLockStudy

2. 重入锁实现：非公平锁lock()过程

![img](https://note.youdao.com/yws/api/personal/file/3048C7A334F44BEFB043C45691FC6C24?method=getImage&version=1290&cstk=6lqG8sux) 

**tryAcquire()**

获取同步状态 state 成功，执行同步代码；<br>
获取同步状态 state 失败，当前线程封装成 Node 节点加入 CLH 队列；<br>
如果 head 为空，CAS创建 header, Node添加到队列尾部；<br>
判断前驱是否为 header, 否则挂起线程，是则自旋获取同步状态 state; 获取失败挂起<br>

**tryRelease()**

CAS设置state - 1；持有线程设为null;<br>
通过LockSupport.unlock()唤醒最前面未被取消的线程;<br>
唤醒之后会进入自旋竞争锁;<br>

可中断锁的不同在于可响应中断。非中断锁判断可中断后设置interrupt标志位true；而可中断锁直接抛出中断异常。

公平所与非公平锁的区别：队列中的线程被唤醒后和新到的线程同时竞争锁，新到的线程可能抢先获取锁，而公平锁会先判断CLH队列中是否有正在等待的线程，如果有自己就排队。



#### 8、Semaphore

Semaphore 类似于操作系统中的信号量，可以控制对互斥资源的访问线程数。

以下代码模拟了对某个服务的并发请求，每次只能有 3 个客户端同时访问，请求总数为 10。

```java
public class SemaphoreExample {

    public static void main(String[] args) {
        final int clientCount = 3;
        final int totalRequestCount = 10;
        Semaphore semaphore = new Semaphore(clientCount);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalRequestCount; i++) {
            executorService.execute(()->{
                try {
                    semaphore.acquire();
                    System.out.print(semaphore.availablePermits() + " ");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                }
            });
        }
        executorService.shutdown();
    }
}
```

```html
2 1 2 2 2 2 2 1 2 2
```
信号量使用：示例源码7：threadsecurity.SemaphoreStudy


#### 9、ReentrantReadWriteLock

读写所示例源码8：threadsecurity.ReadWriteLockStudy

#### 10、CountDownLatch

用来控制一个或者多个线程等待多个线程。

维护了一个计数器 cnt，每次调用 countDown() 方法会让计数器的值减 1，减到 0 的时候，那些因为调用 await() 方法而在等待的线程就会被唤醒。

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/ba078291-791e-4378-b6d1-ece76c2f0b14.png)<br>

```java
public class CountdownLatchExample {

    public static void main(String[] args) throws InterruptedException {
        final int totalThread = 10;
        CountDownLatch countDownLatch = new CountDownLatch(totalThread);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("run..");
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        System.out.println("end");
        executorService.shutdown();
    }
}
```

```html
run..run..run..run..run..run..run..run..run..run..end
```

倒计数器示例源码9：threadsecurity.CountDownLatchStudy

#### 11、CyclicBarrier


用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行。

和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行 await() 方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行。

CyclicBarrier 和 CountdownLatch 的一个区别是，CyclicBarrier 的计数器通过调用 reset() 方法可以循环使用，所以它才叫做循环屏障。

CyclicBarrier 有两个构造函数，其中 parties 指示计数器的初始值，barrierAction 在所有线程都到达屏障的时候会执行一次。

```java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

public CyclicBarrier(int parties) {
    this(parties, null);
}
```

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/f71af66b-0d54-4399-a44b-f47b58321984.png)<br>

```java
public class CyclicBarrierExample {

    public static void main(String[] args) {
        final int totalThread = 10;
        CyclicBarrier cyclicBarrier = new CyclicBarrier(totalThread);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("before..");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
                System.out.print("after..");
            });
        }
        executorService.shutdown();
    }
}
```

```html
before..before..before..before..before..before..before..before..before..before..after..after..after..after..after..after..after..after..after..after..
```
循环栅栏示例源码10：threadsecurity.CyclicBarrierStudy
 

## 五、JAVA并发容器

1. 通过 Collections.synchronizedXXX(xxx) 将 xxx 容器包装为一个线程安全的容器。例如：Collections.synchronizedMap(new HashMap); 会返回一个 SynchronizedMap 对象，其内部实现是封装了传入的 HashMap, 并定义一个 mutex，在原有 HashMap 的每个方法执行时通过 Synchronized 锁定 mutex。


2. ConcurrentHashMap：


3. CopyOnWriteList：

 
4. ConcurrentLinkedList：

 
5. BlockingQueue：暂时不了解详细实现原理

java.util.concurrent.BlockingQueue 接口有以下阻塞队列的实现：

-  **FIFO 队列**  ：LinkedBlockingQueue、ArrayBlockingQueue（固定长度）
-  **优先级队列**  ：PriorityBlockingQueue

提供了阻塞的 take() 和 put() 方法：如果队列为空 take() 将阻塞，直到队列中有内容；如果队列为满 put() 将阻塞，直到队列有空闲位置。

**使用 BlockingQueue 实现生产者消费者问题**  

```java
public class ProducerConsumer {

    private static BlockingQueue<String> queue = new ArrayBlockingQueue<>(5);

    private static class Producer extends Thread {
        @Override
        public void run() {
            try {
                queue.put("product");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("produce..");
        }
    }

    private static class Consumer extends Thread {

        @Override
        public void run() {
            try {
                String product = queue.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("consume..");
        }
    }
}
```

```java
public static void main(String[] args) {
    for (int i = 0; i < 2; i++) {
        Producer producer = new Producer();
        producer.start();
    }
    for (int i = 0; i < 5; i++) {
        Consumer consumer = new Consumer();
        consumer.start();
    }
    for (int i = 0; i < 3; i++) {
        Producer producer = new Producer();
        producer.start();
    }
}
```

```html
produce..produce..consume..consume..produce..consume..produce..consume..produce..consume..
```

5.1. ArrayBlockingQueue：


5.2. LinkedBlockingQueue：


5.3. PriorityBlockingQueue：


## 六、线程池

#### 1、为什么需要线程池

线程的创建和销毁需要一定的开销，如果我们为每一个任务创建一个新的线程来执行的话，那么这些线程的创建与销毁将消耗大量的计算资源。如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

在线程池中创建若干条线程，当有任务需要执行时就从该线程池中获取一条线程来执行任务，如果一时间任务过多，超出线程池的线程数量，那么后面的线程任务就进入一个等待队列进行等待，直到线程池有线程处于空闲时才从等待队列获取要执行的任务进行处理。

 

#### 2、线程池框架Executor

**1. Executor框架两级调度模型**

在上层，java多线程程序通过把应用分为若干个任务，然后使用用户级的调度器（Executor框架） 将这些任务映射为固定数量的线程；在底层，操作系统内核将这些线程映射到硬件处理器上。每一个java线程都会被一对一映射为本地操作系统的线程，操作系统会调度所有的线程并将它们分别给可用的CPU。

**2. Executor框架结构**

- 任务：包括被执行任务需要实现的接口：Runnable接口或Callable接口

- 任务的执行：包括任务执行机制的核心接口Executor，以及继承自Executor的EexcutorService接口。Exrcutor有两个关键类实现了ExecutorService接口（ThreadPoolExecutor和ScheduledThreadPoolExecutor. 。

- 异步计算的结果：包括接口Future和实现Future接口的FutureTask类

    ![img](https://note.youdao.com/yws/api/personal/file/91CF12C64BAA4F09B55FBDAB65294223?method=getImage&version=1293&cstk=6lqG8sux) 

    1. 创建任务：主线程首先创建实现Runnable或Callable接口的任务对象，
    
    2. 提交给线程池：把Runnable对象直接提交给ExecutorService执行，方法为ExecutorService.execute(Runnable command)；或者也可以把Runnable对象或者Callable对象提交给ExecutorService执行，方法为ExecutorService.submit(Runnable task)或ExecutorService.submit(Callable<T> task)。
    
    3. 结果返回：如果执行ExecutorService.submit(...),ExecutorService将返回一个实现Future接口的对象（其实就是FutureTask. 。当然由于FutureTask实现了Runnable接口，我们也可以直接创建FutureTask，然后提交给ExecutorService执行。

 

#### 3、ThreadPoolExecutor构造参数解析

newFixedThreadExecutor(), newCacheThreadExecutor(), newSingleThreadExecutor()内部都调用ThreadPoolExecutor，所以先看看这个类的构造函数如下：

```java
public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,

                long keepAliveTime,TimeUnit unit,
    
                BlockingQueue<Runnable> workQueue) {
    
     this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,Executors.defaultThreadFactory(), defaultHandler);

}
```
1. corePoolSize：线程池的核心线程数，默认情况下，核心线程数会一直在线程池中存活，即使它们处理闲置状态。如果将ThreadPoolExecutor的allowCoreThreadTimeOut属性设置为true，那么闲置的核心线程在等待新任务到来时会执行超时策略，这个时间间隔由keepAliveTime所指定，当等待时间超过keepAliveTime所指定的时长后，核心线程就会被终止。

2. maximumPoolSize：线程池所能容纳的最大线程数量，当活动线程数到达这个数值后，会触发拒绝策略。

3. keepAliveTime：非核心线程闲置时的超时时长，超过这个时长，非核心线程就会被回收。当ThreadPoolExecutor的allowCoreThreadTimeOut属性设置为true时，keepAliveTime同样会作用于核心线程。

4. unit：用于指定keepAliveTime参数的时间单位，这是一个枚举，常用的有TimeUnit.MILLISECONDS(毫秒)，TimeUnit.SECONDS(秒)以及TimeUnit.MINUTES(分钟)等。

5. workQueue：线程池中的任务队列，通过线程池的execute方法提交可执行对象会存储在这个队列中。

6. threadFactory：线程工厂，为线程池提供创建新线程的功能。ThreadFactory是一个接口，它只有一个方法：Thread newThread（Runnable r. 。

7. handler：拒绝策略。包括四种拒绝策略

    ![img](https://note.youdao.com/yws/api/personal/file/0F7AF9A2133D44CA97CFFD1EE32082B0?method=getImage&version=1292&cstk=6lqG8sux) 

#### 4、Executor.execute源码解析

**1. execute**
```java
 public void execute(Runnable command) {

     if (command == null)
    
       throw new NullPointerException();

int c = ctl.get();	//活动线程数

①    if (workerCountOf(c) < corePoolSize) {

       if (addWorker(command, true))
    
         return;
    
       c = ctl.get();
    
     }

②    if (isRunning(c) && workQueue.offer(command)) {

       int recheck = ctl.get();
    
       if (! isRunning(recheck) && remove(command))
    
         reject(command);
    
       else if (workerCountOf(recheck) == 0)
    
         addWorker(null, false);
    
     }

③    else if (!addWorker(command, false))

④      reject(command);

  }
```
- 如果小于 corePoolSize，将通过 addWorker() 方法调度执行。

- 如果大于核心池大小，那么就提交到等待队列。

- 如果进入等待队列失败，则会将任务直接提交给线程池。

- 线程数量已经达到线程池规定的最大值，那么就会拒绝执行此任务，ThreadPoolExecutor 会调用 RejectExecutionHandler 的 rejectExecution 方法来通知调用者。

**2. addWork**

新建并启动Worker

- 判断线程池当前是否为可以添加worker线程的状态，可以则继续下一步，不可以return false：

- 线程池当前线程数量是否超过上限（corePoolSize 或 maximumPoolSize） ，超过了return false，没超过则对workerCount+1，继续下一步

- 在线程池的ReentrantLock保证下，向 WorkersSet 中添加新创建的 worker 实例，添加完成后解锁，并启动 worker 线程，如果这一切都成功了 return true，如果添加worker入Set失败或启动失败，调用addWorkerFailed()逻辑

**3. Worker**

Worker类本身既实现了Runnable，又继承了 AQS ，所以其既是一个可执行的任务，又可以达到锁的效果。封装用户提交的task对象，调用runWork()执行task.run(), 执行之后通过getTask()从workQueue中获取任务继续执行。

1. new Worker()

- 将AQS的state置为-1，在runWoker()前不允许中断

- 待执行的任务会以参数传入，并赋予firstTask

- 用Worker这个Runnable创建Thread

2. runWorker(Worker w)

- Worker线程启动后，通过Worker类的run()方法调用runWorker(this)

- 执行任务之前，首先worker.unlock()，将AQS的state置为0，允许中断当前worker线程

- 开始执行firstTask，调用task.run()，在执行任务前会上锁wroker.lock()，在执行完任务后会解锁，为了防止在任务运行时被线程池一些中断操作中断

- 在任务执行前后，可以根据业务场景自定义beforeExecute() 和 afterExecute()方法

- 无论在beforeExecute()、task.run()、afterExecute()发生异常上抛，都会导致worker线程终止，进入processWorkerExit()处理worker退出的流程

- 如正常执行完当前task后，会通过getTask()从阻塞队列中获取新任务，当队列中没有任务，且获取任务超时，那么当前worker也会进入退出流程

3. getTask()

- 首先判断是否可以满足从workQueue中获取任务的条件，不满足return null

- 如果满足获取任务条件，根据是否需要定时获取调用不同方法：

  - workQueue.poll()：如果在keepAliveTime时间内，阻塞队列还是没有任务，返回null

  - workQueue.take()：如果阻塞队列为空，当前线程会被挂起等待；当队列中有任务加入时，线程被唤醒，take方法返回任务

- 在阻塞从workQueue中获取任务时，可以被interrupt()中断，代码中捕获了InterruptedException，重置timedOut为初始值false，再次执行第1步中的判断，满足就继续获取任务，不满足return null，会进入worker退出的流程

 

#### 4、常用线程池

**1. newFixedThreadExecutor**

```java
  public static ExecutorService newFixedThreadPool(int nThreads) {

     return new ThreadPoolExecutor(nThreads, nThreads,
        0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue());
  }
```

- 可以看出newFixedThreadExecutor内部调用ThreadPoolExecutor，且保证核心线程数和最大线程数相同。没有非核心线程，keepAlive值为0毫秒。所以如果当前运行线程数少corePoolSize，则创建一个新的线程来执行任务。

- 如果当前线程池的运行线程数等于 corePoolSize，那么后面提交的任务将加入 LinkedBlockingQueue。FixedThreadPool 使用的是无界队列 LinkedBlockingQueue 作为线程池的工作队列（队列容量为 Integer.MAX_VALUE） 。由于使用无界队列，运行中的 FixedThreadPool 不会拒绝任务（当然此时是未执行 shutdown 和 shutdownNow 方法） ，所以不会去调用RejectExecutionHandler的rejectExecution方法抛出异常。

- 线程会循环从LinkedBlockingQueue获取任务来执行。

**2. SingleThreadPool**

核心线程数为1的FixedThreadPool

**3. newCachedThreadExecutor**
```java
public static ExecutorService newCachedThreadPool() {

 return new ThreadPoolExecutor(0, Integer.MAX_VALUE,

                60L, TimeUnit.SECONDS,

                new SynchronousQueue<Runnable>());
}
```
- CachedThreadExecutor仍然使用ThreadPoolExecutor，其核心线程数为0，最大线程数是最大整数。且非核心线程60s处于闲置状态就会被销毁。

- 使用直接提交队列（SynchronousQueue） ：SynchronousQueue 是一个特殊的队列，没有容量。CachedThreadExecutor 提交任务时首先执行 SynchronousQueue.offer(Runnable task)，添加一个任务。如果当前 CachedThreadPool 中有空闲线程正在执行 SynchronousQueue.poll(keepAliveTime,TimeUnit.NANOSECONDS)，那么主线程执行ooffer操作与空闲线程执行poll操作配对成功，主线程把任务交给空闲线程执行，execute()方法执行完成。
如果提交失败，那么会直接创建一个Worker执行当前任务。

- 当CachedThreadPool初始线程数为空时，或者当前没有空闲线程，将没有线程去执行SynchronousQueue.poll(keepAliveTime,TimeUnit.NANOSECONDS)。此时CachedThreadPool会创建一个新的线程来执行任务，execute()方法执行完成。

- 创建的新线程将任务执行完成后，会执行 SynchronousQueue.poll(keepAliveTime,TimeUnit.NANOSECONDS)，这个 poll 操作会让空闲线程最多在 SynchronousQueue 中等待60秒，如果60秒内主线程提交了一个新任务，那么这个空闲线程将会执行主线程提交的新任务，否则，这个空闲线程将被终止。由于空闲60秒的空闲线程会被终止，因此长时间保持空闲的 CachedThreadPool 是不会使用任何资源的。

所以，CachedThreadPool 每个插入操作必须等到一个线程与之对应。CachedThreadPool 使用 SynchronousQueue，把主线程的任务传递给空闲线程执行。

**4. 各ThreadPoolExecutor适用场景**

FixedThreadPool：适用于为了满足资源管理需求，而需要限制当前线程的数量的应用场景，它适用于负载比较重的服务器。

SingleThreadExecutor：适用于需要保证执行顺序地执行各个任务；并且在任意时间点，不会有多个线程是活动的场景。

CachedThreadPool：大小无界的线程池，适用于执行很多的短期异步任务的小程序，或者负载较轻的服务器。

#### 5、ScheduledThreadPoolExecutor

继承自 ThreadPoolExecutor，它主要用来在给定的延迟之后执行任务，或者定期执行任务。ScheduledThreadPoolExecutor 可以在构造函数中指定多个对应的后台线程数。
```java
public ScheduledThreadPoolExecutor(int corePoolSize) {

     super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,new DelayedWorkQueue());

}
```
DelayQueue是一个无界队列

- 当调用 ScheduledThreadPoolExecutor 的 schedule() 方法、scheduleAtFixedRate() 方法或者 scheduleWithFixedDelay() 方法时，会向 ScheduledThreadPoolExecutor 的 DelayQueue 添加一个实现了 RunnableScheduledFuture 接口的 ScheduleFutureTask。

- 线程池中的线程从 DelayQueue 中获取 ScheduleFutureTask，然后执行任务。

- schedule 返回一个 ScheduledFuture 对象，用于在给定时间对任务进行一次调度

    `public ScheduledFuture<?> schedule(Runnable command,long delay,TimeUnit unit)`

- scheduleAtFixedRate 方法的作用是预定在初始的延迟结束后，周期性地执行给定的任务，周期长度为 period，其中 initialDelay 为初始延迟。

    `public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,long initialDelay, long period,TimeUnit unit)`

- scheduleWithFixedDelay  方法的作用是预定在初始的延迟结束后周期性地执行给定任务，在一次调用完成和下一次调用开始之间有长度为 delay 的延迟，其中 initialDelay 为初始延迟。

    `public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,long initialDelay,long delay,TimeUnit unit)`

- SingleThreadScheduledExecutor 是大小为 1 的 ScheduledThreadExecutor

- ScheduledThreadPoolExecutor 和 SingleThreadScheduledExecutor 的适用场景

    - ScheduledThreadPoolExecutor：适用于多个后台线程执行周期性任务，同时为了满足资源管理的需求而需要限制后台线程数量的应用场景。
    
    - SingleThreadScheduledExecutor：适用于需要单个后台线程执行周期任务，同时需要保证任务顺序执行的应用场景。

线程池示例源码：threadpool.ThreadPoolStudy

 

#### 6、Future模式

为什么会有Future模式？我们请求的某些业务可能比较耗时，那么我们希望在处理这些业务的时候，调用者可以执行其他的逻辑而不是一直等待。等到业务处理完之后我们在来货物结果。

对应上面的需求，我们首先有一个可执行的任务用于处理业务（这个可执行对象就是一个 Callable） ，当我们请求处理业务的时候，会立即得到一个 Future（这个 Future 通常还没有包含业务处理的结果) ；然后调用者可以先执行其他逻辑，而 Callable 会默默的处理业务，等到业务处理完之后我们可以通过 Future 来获取处理结果。

**1. Callable接口**

从接口我们可以知道用实现Callable接口的可执行对象不同于实现Runnable的可执行对象，因为call()可以有返回值，可以抛出异常。
```java
public interface Callable<V> {

  V call() throws Exception;

}
```
**2. Future接口**

Future 提供了3种功能：中断执行中的任务，判断任务是否执行完成，获取任务执行完成后的结果。
```java
public interface Future<V> {

  boolean cancel(boolean mayInterruptIfRunning);

  boolean isCancelled();

  boolean isDone();

  V get() throws InterruptedException, ExecutionException;

  V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}
```
 
**3. FutureTask**

FutureTask 实现了 Future 接口和 Runnable 接口。可以看见再 FutureTask 中有 Callable，run() 方法会调用 Callable 的 call() 方法得到返回值
```java
public FutureTask(Callable<V> callable) {

     if (callable == null)
    
       throw new NullPointerException();
    
     this.callable = callable;
    
     this.state = NEW;    // ensure visibility of callable

}

public FutureTask(Runnable runnable, V result) {

     this.callable = Executors.callable(runnable, result);
    
     this.state = NEW;    // ensure visibility of callable

}
```
V get() ：获取异步执行的结果，如果没有结果可用，此方法会阻塞直到异步计算完成。

V get(Long timeout , TimeUnit unit) ：获取异步执行结果，如果没有结果可用，此方法会阻塞，但是会有时间限制，如果阻塞时间超过设定的 timeout 时间，该方法将抛出异常。

boolean isDone() ：如果任务执行结束，无论是正常结束或是中途取消还是发生异常，都返回true。

boolean isCanceller() ：如果任务完成前被取消，则返回true。

boolean cancel(boolean mayInterruptRunning) ：如果任务还没开始，执行 cancel(...) 方法将返回 false；如果任务已经启动，执行 cancel(true) 方法将以中断执行此任务线程的方式来试图停止任务，如果停止成功，返回 true；当任务已经启动，执行 cancel(false) 方法将不会对正在执行的任务线程产生影响(让线程正常执行到完成)，此时返回 false；当任务已经完成，执行 cancel(...) 方法将返回 false。mayInterruptRunning 参数表示是否中断执行中的线程。

4. Future模式示例程序：threadpool.FutureModeStudy