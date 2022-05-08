# RocketMQ 消息存储


## 前    言

大家好，我是三此君，一个在自我救赎之路上的非典型程序员。

“一张图”系列旨在通过“一张图”系统性的解析一个板块的知识点：

- 三此君向来不喜欢零零散散的知识点，通过一张图将零散的知识点连接起来，能够让我们对一个板块有更深入、更系统的理解。
- 同时本系列尽可能的精炼，希望能够让大家花 20%的时间，快速理解这个板块下 80% 的内容。

本文是“一张图”系列的第一个板块：一张图解析 RocketMQ。

- 为了叙述的方便，绘图的时候将整个系列分为许多小的模块，讲解的时候也是按照模块循序渐进的。[一张图解析 RocketMQ 原图](https://sm.ms/image/vPtlGbaqHhJ9Tcg)
- 一张图解析 RocketMQ 是会深入到源码层面，但是文中不会粘贴源码。三此君在看源码的时候写了很多备注，可以降低大家看源码的难度，需要的同学自行到三此君的仓库中 Fork：[rocketmq release-4.3.0](https://github.com/sancijun/rocketmq/tree/release-4.3.0)

![一张图进阶 RocketMQ](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508151240.jpeg)

本文是《一张图解析 RocketMQ》系列的第 3 篇，之前我们已经了解的 [RocketMQ 概述]()、[RocketMQ 生产者]()，生产者通过网络传输消息，现在接力棒已经交给了 Broker，那 Broker 是如何存储消息的呢？为什么 RocketMQ 可以有百万的吞吐量呢？要知道 Broker 如何存储消息，我们需要先了解 RocketMQ 的存储结构，也就是消息是如何组织的。了解了存储结构，我们才能更好的理解存储流程，不然我们不知道为什么流程是这样的。最后我们需要了解有哪些机制支撑 RocketMQ 百万吞吐量。

## 存储架构

![RocketMQ 物理存储架构](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508152714.png)

消息在 Broker 上的存储结构如上图，所有的所有相关文件放在 ROCKETMQ_HOME 下，有哪些文件呢？存放消息本身的 CommitLog，以及消息的索引文件 ConsumeQueue 和 IndexFile：

- **CommitLog**

从物理结构上来看，所有的消息都存储在 CommitLog 里面，其实就是所有的消息按照“消息在 CommitLog 各字段示意图”所示，挨个按顺序存储到文件中。

单个 CommitLog 文件大小默认 1G ，文件名长度为 20 位，左边补零，剩余为起始偏移量。比如 00000000000000000000 代表了第一个文件，起始偏移量为 0，文件大小为 1G=1073741824；当第一个文件写满了，第二个文件为 00000000001073741824，起始偏移量为 1073741824，以此类推。消息主要是顺序写入日志文件，当文件满了，写入下一个文件。CommitLog 顺序写，可以大大提高写入效率。

但是问题来了，消息发送的时候我们指定了 Topic，现在所有 Topic 都顺序个写入到 CommitLog，存入的时候是安逸了（顺序写），但是获取消息可就麻烦了。如果我要获取某个 Topic 的消息，需要遍历 commitlog 文件，根据 topic 过滤消息。这个渣男，只管自己爽。有什么办法可以提高消息查询效率呢？

- **ConsumeQueue**

我们再回忆一下，消息存入的时候是指定了 Topic，同时我们也说了每个 Topic 默认创建 4 个 ConsumeQueue（ queueId 标识）。关键就在 ConsumeQueue 上，ConsumeQueue 是指定 Topic 消息的索引文件，怎么理解呢？从“消息在 ConsumeQueue 各字段示意图”可知，每个条目共 20 个字节，分别为 8 字节的 commitlog 物理偏移量、4 字节的消息长度、8 字节 tag hashcode，单个文件由 30W 个条目组成，**可以像数组一样随机访问每一个条目**，每个 ConsumeQueue 文件大小约 5.72M。**ConsumeQueue 文件可以看成是基于 topic 的 commitlog 索引文件**。Consumer 即可根据 ConsumeQueue 来查找待消费的消息。

因为 ConsumeQueue 里只存偏移量信息，所以尺寸是有限的，在实际情况中，**大部分的 ConsumeQueue 能够被全部读入内存，所以这个中间结构的操作速度很快，可以认为是内存读取的速度**。此外为了保证 CommitLog 和 ConsumeQueue 的一致性，CommitLog 里存储了 ConsumeQueues、Message Key、Tag 等所有信息，即使 ConsumeQueue 丢失，也可以通过 CommitLog 完全恢复出来。

ConsumeQueue 文件夹的组织方式如下：topic/queue/file 三层组织结构，具体存储路径为：$HOME/store/consumequeue/{topic}/{queueId}/{fileName}。

- **IndexFile**

IndexFile 是另一种可选索引文件，提供了一种可以通过 key 或时间区间来查询消息的方法。 IndexFile 索引文件其底层实现为 hash 索引，类似于 Java 1.7 HashMap，计算 Key 的 hashcode，hashcode 取余得到 hash 槽，拉链法解决哈希冲突。<br>
Index 文件的存储位置是：$HOME \store\index\${fileName}，文件名 fileName 是以创建时的时间戳命名的，固定的单个 IndexFile 文件大小约为 400M，一个 IndexFile 可以保存 2000W 个索引。

所以，RocketMQ 消息存储架构主要有 CommitLog，ConsumeQueue，IndexFile 构成。我们发送一条消息，会先格式化成“消息在 CommitLog 各字段示意图”中的样子，顺序写入 CommitLog 中，然后 Broker 会按照 ”消息在 ConsumeQueue 各字段示意图“所示构建一条索引记录，存入该消息所属 Topic 的 ConsumeQueue 索引文件中。如果有 IndexFile，还会构建 IndexFile。

现在我们已经知道了 RocketMQ 消息的存储结构，接下来我们的就要了解 RocketMQ 是如何构建 CommitLog、ConsumeQueue 和 IndexFile，以及 RocketMQ 如何保证性能支撑百万吞吐量的？这是本文的主要目标，一定要抓住主要目标，不要走丢咯。

## 启动流程

了解了 RocketMQ 消息在磁盘中是怎么存储的，我们就可以来看看具体的存储流程了。首先，还是先来看看 Broker 的启动流程。初始化过程都是这个鸟样，只看初始化过程完全不知所云，但是不看初始化过程，直接看具体执行流程也是摸不着头脑，一堆组件不知道从哪里来的，所以我们还是先耐着性子大致看看。但这并不是我们关注的重点，注意几个关键点即可。

![Broker 启动流程](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508152023.png)

- 初始化启动环境。部署好 RocketMQ 后，执行/bin/mqbroker 脚本，主要用于设置 RocketMQ 目录环境变量，例如 ROCKETMQ_HOME 。然后调用 ./bin/runbroker.sh 进入 RocketMQ 的启动入口，主要设置了 JVM 启动参数，比如 JAVA_HOME、Xms、Xmx。执行 main 函数。

- 初始化 BrokerController。该初始化主要包含 RocketMQ 启动命令行参数解析、**NettyRemotingServer 初始化**、Broker 各个模块配置参数解析、Broker 各个模块初始化、进程关机 Hook 初始化等过程。

- 启动 RocketMQ 的各个组件。但是这些组件并不是每一个都是核心组件，部分组件会在后面的流程中使用，这里混个眼熟，如果后面流程没有提及的大家可以暂且跳过，我们的目标是把握 RocketMQ 的核心内容，而不是每个细节。

  > MessageStore：存储层服务，比如 **CommitLog、ConsumeQueue 存储管理，消息刷盘，索引构建**等。
  > **RemotingServer**：普通通道请求处理服务。一般的请求都是在这里被处理的。
  > FastRemotingServer：VIP 通道请求处理服务。如果普通通道比较忙，那么可以使用 VIP 通道，一般作为客户端降级使用。
  > BrokerOuterAPI：Broker 访问对外接口的封装对象。
  > PullRequestHoldService：Pull 长轮询服务。
  > ClientHousekeepingService：清理心跳超时的生产者、消费者、过滤服务器。
  > FilterServerManager：过滤服务器管理。

## 存储流程

![Broker 存储流程](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508152122.png)

在前面 RocketMQ 存储结构中我们了解了 RocketMQ 将所有消息顺序写入 CommitLog，然后构建 ConsumeQueue/IndexFile 索引文件，所以这个小结我们主要的目标就是看看这些文件是如何构建的。

- Broker 启动流程中很关键的一点是启动了 NettyRemotingServer，在 [RocketMQ 生产者]( 中我们介绍过 RocketMQ 的通信机制 NettyRemotingServer 初始化会监听端口等待客户端连接，当客户端发送请求的时，NettyRemotingServer WorkerGroup 处理可读事件，调用 NettyServerHandler.channelRead0() 处理数据。

  接着调用链到 processRequestCommand 方法，这个方法主要是根据请求中的 RequestCode，从本地缓存 processorTable 中获取相应的 Processor 来执行后续逻辑。处理器是什么？处理器的缓存从哪里来？

  > Processor 就是用来处理特定请求的执行者，例如，生产者存入消息使用 SendMessageProcessor，查询消息使用 QueryMessageProcessor，拉取消息使用 PullMessageProcessor。在 Broker 启动流程中有一步是注册 Processor，以 RequestCode 为 Key ，Processor 为值，添加到 processorTable 缓存中。

  接着 [RocketMQ 生产者]( 消息发送流程来看，当生产者的请求达到 Broker，Broker 获取的 Processor 应为 SendMessageProcessor。封装一个 Runable 对象，run 方法内调用 SendMessageProcessor.processRequest ，提交到线程池，继续后面的处理。

- SendMessageProcessor.processRequest 调用 sendMessage 方法，主要包含消息的校验及重试逻辑处理，然后调用存储模块 DefaultMessageStore 存储消息。

  > 消息校验：校验 Broker 是否配置可写，校验 Topic 名字是否为默认值，获取或创建 topicConfig，判断 queueId 是否超过限制。
  >
  > 重试消息处理：消费者消费失败后会将消息发回给 Broker，这里我们暂且认为就是生产者发送的请求，先看下面的流程。

- DefaultMessageStore.putMessage 只是做了很多的校验，简单看看即可。包括：如果当前 Broker 停止工作则拒绝消息写入、Broker 为 SLAVE 角色则拒绝消息写入、当前 RocketMQ 不支持写入则拒绝消息写入、主题长度超过 256 个字符则拒绝消息写入、消息属性长度超过 65536 个字符则拒绝消息写入、PageCache 忙则报错。然后调用 CommitLog.putMessage 存入消息。

- 看到这里应该稍微熟悉一些了，终于到我们期待已久的 CommitLog 出场了。主要是延迟消息处理，然后获取可以写入的 CommitLog 进行写入。

  >延迟消息处理：如果消息的延迟级别大于 0，将消息的原主题名称与原消息队列 ID 存入消息属性中，用延迟消息主题 SCHEDULE_TOPIC、消息队列 ID 更新原先消息的主题与队列，这是并发消息消费重试关键的一步。但不是这个本节的主要目标，后文会进一步分析。

  关键点在如何获取可以写入的 CommitLog。存储结构小节里面有提到每个 CommitLog 默认大小 1G，写完一个文件，以偏移量命名创建下一个文件。每个 1G 大小 CommitLog 的在代码层面对应的是 MappedFile，而多个 MappedFiled 组成 MappedFileQueue。逻辑上的 CommitLog 通过持有 MappedFileQueue 管理多个 MappedFile。所以，获取可以写入的 CommitLog 也就是获取 MappedFileQueue 最后一个 MappedFile，为什么是最后一个，因为前面的已经写完了呀。来看看 RocketMQ 逻辑与物理存储的对应关系应该能够更直观的理解。

  ![逻辑-物理存储结构](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508152208.png)

- 获取到最后一个 MappedFile 后，调用 MappedFiled.appendMessage 将消息追加到该文件中。可是尽管是顺序写入，但是连小学生都知道写磁盘还是很慢，难道想这样支撑 RocketMQ 百万吞吐量？too young too simple！

  从逻辑存储结构和物理存储结构的映射关系来看，MappedFile 持有物理 CommitLog 的 fileChannel (Java NIO 文件读写的通道)，通过 fileChannel 可以访问物理 CommitLog 文件，但是 RocketMQ 并没有直接使用 fileChannel，而是映射到一个 MappedByteBuffer，我们的目的就是把消息写入这个 ByteBuffer 中，进而写入 MappedFile 对应的 CommitLog 文件。为什么需要这样做，还有哪些细节，会在”文件内存映射“小结中为大家解答。

- 继续看流程，得到 MappedFile 对应的 ByteBuffer，我们需要将消息序列化，写入 ByteBuffer 中。

  > 1. 构建消息 id, createMessageId
  > 2. 获取该消息在消息队列的偏移量，CommitLog 中保存了当前所有消息队列的当前待写入偏移量。
  > 3. 判断是否是事务消息：这里主要处理 Prepared 类型和 Rollback 类型的消息，设置消息 queueOffset 为 0
  > 4. 计算消息总大小，calMsgLength。
  > 5. 判断文件的剩余空间，是否足够写入当条消息，如果不可以，则将文件末尾写入剩余空间大小+固定魔数；然后返回一个 END_OF_FILE 的结果
  > 6. 如果空间足够，这将这条消息写入之前得到的 MappedFile 的 ByteBuffer 中。
  > 7. 将各字段按照”消息在 CommitLog 各字段示意图“存入 Bytebuffer，然后返回 PUT_OK 结果

## 文件内存映射

我们已经知道 MappedFile 持有 CommitLog 文件的 fileChannel，可以通过 fileChannel 访问 CommitLog，但是 RocketMQ 却没有使用 fileChannel 访问 CommitLog，而是映射到一个 MappedByteBuffer ？这里就有疑问了，映射是什么意思？什么是 MappedByteBuffer，有什么用，为什么要这样做？

这里的映射其实是使用 Java NIO 的内存映射 Buffer，将文件映射到内存中，得到 MappedByteBuffer。就是传说中的**文件内存映射**，映射文件同时具有内存的写入速度和与磁盘一样可靠的持久化方式。我们都知道磁盘 I/O 速度非常慢，文件内存映射就是 RockerMQ 支撑百万吞吐量的关键之一，可为什么文件内存映射会有那么大的威力？我们需要先来简单回顾一下传统的磁盘 I/O，看看它有什么大病。

### 传统 I/O

Linux 操作系统分为“用户态”和“内核态”，文件操作、网络操作需要涉及这两种形态的切换。一台服务器把本机磁盘文件的内容发送到客户端，一般分为两个步骤：

- read(file, tmp_buf, len)：把数据从存储器 （磁盘、网卡等） 读取到用户缓冲区
- write(socket, tmp_buf, len)：把数据从用户缓冲区写出到存储器

![Linux 传统 I/O](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508152501.png)

可以清楚看到这里一共触发了 4 次用户态和内核态的上下文切换，分别是 `read()/write()` 调用和返回时的切换，2 次 DMA 拷贝，2 次 CPU 拷贝，加起来一共 4 次拷贝操作。通过引入 DMA，我们已经把 Linux 的 I/O 过程中的 CPU 拷贝次数从 4 次减少到了 2 次，但是 CPU 拷贝依然是代价很大的操作，对系统性能的影响还是很大，特别是那些频繁 I/O 的场景，更是会因为 CPU 拷贝而损失掉很多性能，我们需要进一步优化，**减少甚至是完全避免 CPU 拷贝**。

### mmap

>  零拷贝技术是指计算机执行操作时，[CPU](https://zh.wikipedia.org/wiki/中央处理器）不需要先将数据从某处 [内存](https://zh.wikipedia.org/wiki/随机存取存储器）复制到另一个特定区域。这种技术通常用于通过网络传输文件时节省 CPU 周期和 [内存带宽](https://zh.wikipedia.org/wiki/内存带宽)。

零拷贝可以减少甚至完全避免操作系统内核和用户应用程序地址空间这两者之间进行数据拷贝操作，并且减少用户态 -- 内核态上下文切换带来的系统开销，这些是为什么零拷贝提升 I/O 性能的原因。零拷贝技术有文多种，RocketMQ 主要使用 mmap 技术，而 kafka 主要使用 sendFile，所以我们也主要了解 mmap 技术。

![Linux I/O Mmap](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508152416.png)

利用 `mmap()` 替换 `read()`，配合 `write()` 调用的整个流程如下：

1. 用户进程调用 `mmap()`，从用户态陷入内核态，将内核缓冲区映射到用户缓存区；
2. DMA 控制器将数据从硬盘拷贝到内核缓冲区；
3. `mmap()` 返回，上下文从内核态切换回用户态；
4. 用户进程调用 `write()`，尝试把文件数据写到内核里的套接字缓冲区，再次陷入内核态；
5. CPU 将内核缓冲区中的数据拷贝到的套接字缓冲区；
6. DMA 控制器将数据从套接字缓冲区拷贝到网卡完成数据传输；
7. `write()` 返回，上下文从内核态切换回用户态。

通过使用 mmap 的方式，可以省去向用户态的内存复制，提高速度。这种机制在 Java 中是通过 MappedByteBuffer 实现的，RocketMQ 通过 mmap 方式优化文件读写性能。

RocketMQ 主要通过 MappedByteBuffer 对文件进行读写操作。其中，利用了 NIO 中的 FileChannel 将磁盘上的物理文件直接映射到用户态的内存地址中（这种 mmap 的方式减少了传统 I/O 将磁盘文件数据在操作系统内核地址空间的缓冲区和用户应用程序地址空间的缓冲区之间来回进行拷贝的性能开销），将对文件的操作转化为直接对内存地址进行操作，从而极大地提高了文件的读写效率（正因为需要使用内存映射机制，故 RocketMQ 的文件存储都使用定长结构来存储，方便一次将整个文件映射至内存）。

## PageCache

在存储流程小节的 Broker 逻辑/物理存储结构图中，我们可以发现 MappedFile 持有 fileChannel、mappedByteBuffer、writeBuffer。fileChannel 指向对应的物理文件，并通过 fileChannel 创建 mappedByteBuffer，mappedByteBuffer 就对应上面提到的 **mmap** 文件内存映射。通过 mmap 方式写入文件时，消息并没有直接落到磁盘上，而是先写入操作系统的 PageCache。

顺序读写 PageCache 异步刷盘：页缓存（PageCache) 是 OS 对文件的缓存，用于加速对文件的读写。一般来说，程序对文件进行顺序读写的速度几乎接近于内存的读写速度，主要原因就是由于 OS 使用 PageCache 机制对读写访问操作进行了性能优化，将一部分的内存用作 PageCache。对于数据的写入，OS 会先写入至 PageCache 内，随后通过异步的方式由内核线程将 pageCache 内的数据刷盘至物理磁盘上。

预读取：对于数据的读取，如果一次读取文件时出现未命中 PageCache 的情况，OS 从物理磁盘上访问读取文件的同时，会顺序对其他相邻块的数据文件进行预读取。

在 RocketMQ 中，ConsumeQueue 逻辑消费队列存储的数据较少，并且是顺序读取，在 PageCache 机制的预读取作用下，ConsumeQueue 文件的读性能几乎接近读内存，即使在有消息堆积情况下也不会影响性能。而对于 CommitLog 消息存储的日志数据文件来说，读取消息内容时候会产生较多的随机访问读取，严重影响性能。如果选择合适的系统 I/O 调度算法，比如设置调度算法为“Deadline”（此时块存储采用 SSD 的话），随机读的性能也会有所提升。

上面只提到了 mappedByteBuffer，那writeBuffer又是什么呢？writeBuffer 在 transientStorePoolEnable 为 true 启用，是通过 ByteBuffer 分配**直接内存，并锁定在内存中（不换到虚拟内存）**

> Tips：文件内存映射、零拷贝、PageCache 都和操作系统密切相关，三此君在这里只是简单的介绍，保证大家能够更好的理解 RocketMQ，感兴趣的同学可以进行深入的了解。也可以留言给三此君，三此君会尽快安排相关内容的分享。

## 消息刷盘

从上面的流程可以看到，目前还只是写入内存中，这部分的速度都是很快的。那 RocketMQ 什么时候将消息写入磁盘呢？这就涉及到我们消息的刷盘机制，也就是流程中大家看到的 handleDiskFlush。消息刷盘在实现上分为同步刷盘、异步刷盘和异步转存：

- 同步刷盘服务（GroupCommitService）：在 Broker 存储消息到 PageCache 后，同步将 PageCache 刷到磁盘，再返回客户端消息并写入结果。同步刷盘对 MQ 消息可靠性来说是一种不错的保障，但是性能上会有较大影响，一般适用于金融业务应用该模式较多。

- 异步刷盘服务（FlushRealTimeService）：在 Broker 存储消息到 PageCache 后，立即返回客户端写入结果，然后异步刷盘服务将 PageCache 异步刷到磁盘。消息刷盘采用后台异步线程提交的方式进行，降低了读写延迟，提高了 MQ 的性能和吞吐量。

- 异步转存服务（CommitRealTimeService）：Broker 通过配置读写分离将消息写入**直接内存（Direct Memory，简称 DM）**，然后通过异步转存服务，将 DM 中的数据再次存储到 PageCache 中，以供异步刷盘服务将 PageCache 刷到磁盘中。

![消息刷盘](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508152607.png)

- 先是判断同步刷盘还是异步刷盘，同步刷盘则构建并提交同步刷盘请求。否则判断 isTransientStorePoolEnable 是否为 true，false 则直接调用异步刷盘服务，true 则调用异步转存服务。
- 同步刷盘：同步刷盘首先构建刷盘请求提交到同步刷盘请求列表中，然后等待同步刷盘任务完成，如果超时则返回刷盘错误，刷盘成功后正常返回给调用方。
  同步刷盘服务 GroupCommitService 是一个循环线程，run 方法每 10s 执行一次 doCommit，当然如果有同步请求提交会立即执行刷盘。
  doCommit 判断当前消息是否已经刷盘，如果没有刷盘则调用 CommitLog.mappedFileQueue.flush 方法，实际上调用的是 mappedByteBuffer.force 方法进行强制刷盘。刷盘之后唤醒调用线程。
- 异步刷盘：异步刷盘服务 FlushRealTimeService 也是一个循环线程，如果超过 10s 或者大于 4 页数据没有刷盘则执行刷盘，也是调用 CommitLog.mappedFileQueue.flush 方法进行刷盘。
- 异步转存：异步转存服务是在 transientStorePoolEnable 为 true 时，消息并没有写入 PageCache，而是写入直接内存中，异步转存只是将直接内存中的消息提交到 PageCache，然后唤醒异步刷盘服务进行刷盘。

## 索引构建

大家到这里，消息都刷盘了，应该都万事大吉了吧？忘记了我们的主要目标除了 CommitLog 刷盘，还有 ComsumQueue/IndexFile 的构建呢？可是整个流程执行下来没有看到哪里构建了索引文件呀？得往回倒一倒，Broker 启动的时候，我们启动了一个叫 ReputMessageService 服务，这个服务本身是循环线程，专门用来构建索引文件，来看看肿么肥四吧。

![索引构建](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508152644.png)

- ConsumeQueue 和 IndexFile 两个索引都是由 ReputMessageService 线程构建，ReputMessageService 继承 ServiceThread，ReputMessageService 线程每执行一次任务推送休息 1 毫秒就继续尝试调用 doReput() 推送消息到消息消费队列和索引文件。
- 从 CommitLog 中查找未创建索引的消息，将消息组装成 DispatchRequest 对象，该逻辑主要在 CommitLog.checkMessageAndReturnSize() 方法中实现。
- 调用 doDispatch() 方法，调用 CommitLogDispatcherBuildConsumeQueue 和 CommitLogDispatcherBuildIndex 两个索引处理器的 dispatch() 方法来处理 DispatchRequest。CommitLogDispatcherBuildConsumeQueue 索引处理器用于构建 ConsumeQueue，CommitLogDispatcherBuildIndex 用于构建 Index file。ConsumeQueue 是必须创建的，IndexFile 是否需要创建则是通过设置 messageIndexEnable 为 True 或 False 来实现的，默认为 True。
- ConsumeQueue 索引文件的构建，首先要查找或者创建一个 ConsumeQueue，从上面 ”Broker 逻辑上“图所示，ConsumeQueue 的组织方式其实和 CommitLog 是一样的，也是持有 MappedFileQueue，然后获取可写入的 LastMappedFile。依次将消息偏移量、消息长度、tag hashcode 写入到 ByteBuffer 中，并根据 consumeQueueOffset 计算 ConumeQueue 中的物理地址。ConsumeQueue 的刷盘方式也是采用的 CommitLog 一步刷盘方式。
- IndexFile 索引文件组织方式和前面两者不一样，并且是可选的。IndexFile 持有物理文件引用的文件映射内存 MappedByteBuffer，我们只需要按照 ”消息在 IndexFile 各字段示意图“所以写入各个字段即可。IdnexFile 持久化方式也不同，是在 getAndCreateLastIndexFile 会创建线程去刷盘。

## 总    结

以上就是 RocketMQ 消息存储的主要内容，我们简单总结一下：

- 要理解消息的存储流程需要先知道消息的存储结构：在物理上消息挨个顺序写入 CommitLog，为了提升消息查询效率需要构建消息的索引文件 ConsumeQueue/IndeFile；
- Broker 启动时进行参数解析，并初始化了 NettyRemotingServer，启动存储服务用于消息存储及索引构建等；
- Broker 收到消息存储请求，经过层层校验，获取 CommitLog 对应的 MappedFile，将消息写入MappedFile 对应的内存映射ByteBuffer；(如果开启了isTransientStorePoolEnable，先写入 DM，再转存内存映射 ByteBuffer)
- 写入 ByteBuffer 还不行，消息要落在磁盘上才不会丢失。消息刷盘分为同步刷盘、异步刷盘和异步转存，异步转存是定时任务将 DM 的消息提交到 MappedByteBuffer 中，再有异步刷盘线程进行刷盘。同步刷盘和异步刷盘最终都是调用CommitLog.mappedFileQueue.flush 方法进行刷盘
- 刷盘之后还需要构建消息对应的索引，索引构建由专门的后台线程，每隔一秒执行一次。
- 最后，最最最重要的就是记得关注，记得点赞，记得转发，记得收藏呀！！！

![](https://cdn.jsdelivr.net/gh/sancijun/images/pics/qrcode_banner.webp)

## 参考文献

- [RocketMQ 官方文档](https://github.com/apache/rocketmq/tree/master/docs/cn)
- [RocketMQ 源码](https://github.com/apache/rocketmq/tree/master)
- [Linux I/O 原理和 Zero-copy 技术全面揭秘](https://mp.weixin.qq.com/s/dZNjq05q9jMFYhJrjae_LA)

- 丁威, 周继锋. RocketMQ技术内幕：RocketMQ架构设计与实现原理. 机械工业出版社, 2019-01.

- 李伟. RocketMQ分布式消息中间件：核心原理与最佳实践. 电子工业出版社, 2020-08.
- 杨开元. RocketMQ实战与原理解析. 机械工业出版社, 2018-06.

