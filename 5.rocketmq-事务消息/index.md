# RocketMQ 事务消息


## 前    言

大家好，我是三此君，一个在自我救赎之路上的非典型程序员。

“一张图”系列旨在通过“一张图”系统性的解析一个板块的知识点：

- 三此君向来不喜欢零零散散的知识点，通过一张图将零散的知识点连接起来，能够让我们对一个板块有更深入、更系统的理解。
- 同时本系列尽可能的精炼，希望能够让大家花 20% 的时间，快速理解这个板块下 80% 的内容。

本文是“一张图”系列的第一个板块：一张图解析 RocketMQ。

- 为了叙述的方便，绘图的时候将整个系列分为许多小的模块，讲解的时候也是按照模块循序渐进的。[一张图解析 RocketMQ 原图](https://sm.ms/image/vPtlGbaqHhJ9Tcg)
- 一张图解析 RocketMQ 是会深入到源码层面，但是文中不会粘贴源码。三此君在看源码的时候写了很多备注，可以降低大家看源码的难度，需要的同学自行到三此君的仓库中 Fork：[rocketmq release-4.3.0](https://github.com/sancijun/rocketmq/tree/release-4.3.0)

![一张图进阶 RocketMQ](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508151240.jpeg)

本文是《一张图解析 RocketMQ》系列的第 5 篇，之前我们已经了解的 [RocketMQ 概述]()、[RocketMQ 生产者]()、[RocketMQ 消息存储]()、[RocketMQ 消费者]()。RocketMQ 大部分内容已经讲完了，可是还有一些很重要的特性得和大家唠唠，今天要分享的就是其中很重要的 RocketMQ 事务消息。

所谓事务消息，其实就是基于 MQ 的分布式事务解决方案。如果直接上来就看事务消息的实现，大家可能觉得云里雾里的。所以，我们还是需要先了解下什么事分布式事务问题。

## 分布式事务问题

事务，我们是上小学就学过了，事务是满足 ACID 性质的一些列操作集合。当然，为了保证事务的 ACID 性质，其实有一套非常复杂的机制。可是残酷的世界从来不会让我们失望，我们连对事务的认知都还停留在 ACID 层面，现在又来一个分布式事务。小小年纪承受了那么多本不应该承受的，也只能硬着头皮上了。

### 什么是分布式事务

我们先来看一个电商下单付款案例：用户完成一次下单并付款，服务器接收请求，首先创建新订单，然后扣减商品库存，最后从用户账户余额扣除金额。这几个操作要么一起成功，要么一起失败。如果我们所有服务都部署在同一台服务器上，并且只有一个数据库，这个问题就是我们小学学过的数据库事务问题。可是实力不允许啊，直接做个淘宝，上微服务、中台、DDD，全都一起上……拍脑袋谁不会，步子太大，恐怕扯着胯。现在分成了订单服务，库存服务和用户服务，完成上面的操作需要访问**三个不同的微服务和三个不同的数据库**。

![](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508153522.png)

这下问题可就大了，我们没办法使用数据库的事务能力了，要是下单成功，没减库存，没扣钱，那岂不是亏大了？当我们把三件事情看做一个事情事，要保证“业务”的一致性，要么所有操作全部成功，要么全部失败，不允许出现部分成功部分失败的现象，这就是分布式系统下的事务了。

**分布式事务是指是指事务各个参与者分别位于分布式系统的不同节点之上，通过网络通信来达到分布式一致性。**网络通信不可避免出现失败、超时的情况，因此分布式事务的实现比本地事务面临更多的困难。概括起来，分布式事务有三种场景：跨数据库分布式事务、跨服务分布式事务、混合式分布式事务。

### 常见解决方案

| 解决方案                     | 描述                                                         |
| ---------------------------- | :----------------------------------------------------------- |
| 基于 XA/2PC 实现的分布式事务 | XA (eXtended Architecture )  是 X/Open 组织提出的一套名为X/Open XA 的处理事务规范，定义了全局的事务管理器（Transaction Manager，用于协调全局事务）和局部的资源管理器（Resource Manager，用于驱动本地事务）之间的通信接口。 |
| 基于 TCC 实现的分布式事务    | TCC (Try、Commit、Cancel) 是一种补偿型事务，该模型要求应用的每个服务提供 try、confirm、cancel 三个接口，它的核心思想是通过对资源的预留（提供中间态），尽早释放对资源的加锁，如果事务可以提交，则完成对预留资源的确认，如果事务要回滚，则释放预留的资源。TCC 也是一种两阶段提交协议，可以看作 2PC/XA 的一种变种，本质是一个应用层面上的 2PC，但是不会长时间持有资源锁。 |
| 基于 Saga 实现的分布式事务   | Saga 并不是一个新概念，其相关论文在 1987 年就发布了，和 XA 两阶段提交规范出现的时间差不多。Saga 和 TCC 一样，也是一种补偿事务，但是它没有 try 阶段，而是把分布式事务看作一组本地事务构成的事务链。<br />事务链中的每一个正向事务操作，都对应一个可逆的事务操作。Saga 事务协调器负责按照顺序执行事务链中的分支事务，分支事务执行完毕，即释放资源。如果某个分支事务失败了，则按照反方向执行事务补偿操作。 |
| 基于事务消息实现的分布式事务 | 基于 MQ 的分布式事务解决方案，适用于对最终一致性敏感度较低的业务场景，例如跨企业的系统间的调用，适用的场景有限。 |

基于 XA 协议的二阶段提交协议方法和三阶段提交协议方法，采用了强一致性，遵从 ACID，基于消息的最终一致性方法，采用了最终一致性，遵从 BASE 理论。之所以有这么多解决方案，是因为任何事情都没有银弹，只有最合适当前场景的解决方案。

各种各样的分布式事务解决方案已经超出了本文的范围，如果大家希望进一步了解分布式事务解决方案，可以吧“安排”打在公屏上，三此君就会尽快为大家安排。本文的目的还是分享 RocketMQ 的事务消息，而事务消息本身是基于 2PC 思想的，故本文只会带大家了解必要的 2PC 相关内容。

### XA/2PC

XA（XA 是 eXtended Architecture 的缩写） 规范中定义了分布式事务处理模型，这个模型中包含三个核心角色：

- RM (Resource Managers)：资源管理器，提供数据资源的操作、管理接口，保证数据的一致性和完整性。最有代表性的就是数据库管理系统，当然有的文件系统、MQ 系统也可以看作 RM。
- TM (Transaction Managers)：事务管理器，是一个协调者的角色，协调跨库事务关联的所有 RM 的行为。
- AP (Application Program)：应用程序，按照业务规则调用 RM 接口来完成对业务模型数据的变更，当数据的变更涉及多个 RM 且要保证事务时，AP 就会通过 TM 来定义事务的边界，TM 负责协调参与事务的各个 RM 一同完成一个全局事务。

XA 规范中分布式事务是构建在 RM 本地事务（此时本地事务被看作分支事务）的基础上的，TM 负责协调这些分支事务要么都成功提交、要么都回滚。

![XA/2PC 示意图](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508153548.png)

XA 规范把分布式事务处理过程划分为两个阶段，所以又叫两阶段提交协议（two phrase commit）：

**1. 预备阶段**

TM 记录事务开始日志，并询问各个 RM 是否可以执行提交准备操作。

RM 收到指令后，评估自己的状态，尝试执行本地事务的预备操作：预留资源，为资源加锁、执行操作等，但是并不提交事务，并等待 TM 的后续指令。如果尝试失败则告知 TM 本阶段执行失败并且回滚自己的操作，然后不再参与本次事务（以 MySQL 为例，这个阶段会完成资源的加锁，redo log 和 undo log 的写入）。

TM 收集 RM 的响应，记录事务准备完成日志。

**2. 提交/回滚阶段**

这个阶段根据上个阶段的协调结果发起事务的提交或者回滚操作。

如果所有 RM 在上一个步骤都返回执行成功，那么：

- TM 记录事务 commit 日志，并向所有 RM 发起事务提交指令。

- RM 收到指令后，提交事务，释放资源，并向 TM 响应“提交完成”。

- 如果 TM 收到所有 RM 的响应，则记录事务结束日志。

如果有 RM 在上一个步骤中返回执行失败或者超时没有应答，则 TM 按照执行失败处理，那么：

- 记录事务 abort 日志，向所有 RM 发送事务回滚指令。

- RM 收到指令后，回滚事务，释放资源，并向 TM 响应回滚完成。

- 如果 TM 收到所有 RM 的响应，则记录事务结束日志。

## RocketMQ 事务消息

### 事务消息流程概述

Apache RocketMQ 在 4.3.0 版中已经支持分布式事务消息，这里 RocketMQ 采用了 2PC 的思想来实现了提交事务消息，所以我们肯定可以在 RocketMQ 事务消息流程中看到预备阶段及提交/回滚两个阶段。同时RocketMQ增加一个补偿逻辑来处理二阶段超时或者失败的消息，如下图所示。

![RocketMQ 事务消息流程](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508153630.png)

**预备阶段**

1. 发送 Half 消息（也可以叫 prepare 消息）。

2. 服务端响应消息写入结果。

3. 根据发送结果执行本地事务（如果写入失败，此时 Half 消息对业务不可见，本地逻辑不执行）。

**提交/回滚阶段**

4. 根据本地事务状态执行 Commit 或者 Rollback（Commit 操作生成消息索引，消息对消费者可见）

**补偿机制**

5. 对没有 Commit/Rollback 的事务消息（pending 状态的消息），从服务端发起一次“回查”

6. Producer 收到回查消息，检查回查消息对应的本地事务的状态

7. 根据本地事务状态，重新 Commit 或者 Rollback

### 事务消息示例

```java
public class TransactionProducer {
   public static void main(String[] args) throws MQClientException, InterruptedException {
       // 实例化事务监听器，主要用于执行本地事务，保存本地事务状态
       TransactionListener transactionListener = new TransactionListenerImpl();
       // 实例化事务消息生产者
       TransactionMQProducer producer = new TransactionMQProducer("please_rename_unique_group_name");
       // 事务消息执行线程池
       ExecutorService executorService = new ThreadPoolExecutor(2, 5, 100, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(2000), new ThreadFactory() {
           @Override
           public Thread newThread(Runnable r) {
               Thread thread = new Thread(r);
               thread.setName("client-transaction-msg-check-thread");
               return thread;
           }
       });
       producer.setExecutorService(executorService);
       producer.setTransactionListener(transactionListener);
       producer.start();
       // 发送消息
       String[] tags = new String[] {"关注", "点赞", "收藏", "转发", "三连呀！"};
       for (int i = 0; i < 10; i++) {
           try {
               Message msg =
                   new Message("TopicTest1234", tags[i % tags.length], "KEY" + i,
                       ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
               SendResult sendResult = producer.sendMessageInTransaction(msg, null);
               System.out.printf("%s%n", sendResult);
               Thread.sleep(10);
           } catch (MQClientException | UnsupportedEncodingException e) {
               e.printStackTrace();
           }
       }
       for (int i = 0; i < 100000; i++) {
           Thread.sleep(1000);
       }
       producer.shutdown();
   }
}
// 实现 TransactionListener，即实现TransactionListener定义的两个方法：executeLocalTransaction和checkLocalTransaction
public class TransactionListenerImpl implements TransactionListener {
  private AtomicInteger transactionIndex = new AtomicInteger(0);
  private ConcurrentHashMap<String, Integer> localTrans = new ConcurrentHashMap<>();
  // 执行本地事务
  @Override
  public LocalTransactionState executeLocalTransaction(Message msg, Object arg) {
      int value = transactionIndex.getAndIncrement();
      int status = value % 3;
      localTrans.put(msg.getTransactionId(), status);
      return LocalTransactionState.UNKNOW;
  }
  // 查询本地事务状态
  @Override
  public LocalTransactionState checkLocalTransaction(MessageExt msg) {
      Integer status = localTrans.get(msg.getTransactionId());
      if (null != status) {
          switch (status) {
              case 0:
                  return LocalTransactionState.UNKNOW;
              case 1:
                  return LocalTransactionState.COMMIT_MESSAGE;
              case 2:
                  return LocalTransactionState.ROLLBACK_MESSAGE;
          }
      }
      return LocalTransactionState.COMMIT_MESSAGE;
  }
}
```

从 RocketMQ 的事务消息示例中，我们可以知道事务消息的入口在 `producer.sendMessageInTransaction`，然后实现了TransactionListener.executeLocalTransaction 用于执行本地事务，TransactionListener.checkLocalTransaction 用于查询本地事务状态。这些方法在什么时候被调用？和我们上面提到的流程有什么关系？又是如何保证分布式事务的正确的？

### 事务消息实现

下图已经总结了我们本文要讲的 RocketMQ 事务消息预提交、提交/回滚及回查的流程及原理，3 个 :star: 标注的地方就是三个阶段的入口，我们分别来看看这三个流程：

![RocketMQ 事务消息实现](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508153717.png)

### 预备阶段

预备阶段主要的目标就是发送 Half 消息并执行本地事务：

- 首先调用了 TransactionMQProducer.sendMessageInTransaction 方法发送事务消息，其实消息的发送存储流程都大同小异。一阶段的消息对用户是不可见的，RocketMQ 将一阶段的 Half 消息存入用户不可见的 RMQ_SYS_TRANS_HALF_TOPIC 主题，这是 RocketMQ 的惯用伎俩之一（延迟消息就是这样）。

- 那么RocketMQ 怎么知道要存入 RMQ_SYS_TRANS_HALF_TOPIC (下文简称 HALF_TOPIC) 还是消息原来的 Topic ？所以我们在发送事务消息的时候要标识这是事务消息。DefaultMQProducerImpl.sendMessageInTransaction 方法中设置 TRAN_MSG = true 和PGROUP，分别表示消息为prepare消息、消息所属消息生产者组。

- DefaultMQProducerImpl.sendKernelImpl 根据 TRAN_MSG 判断是否是事务消息，是则设置 sysFlag 系统标记为MessageSysFlag.TRANSACTION_PREPARED_TYPE，Broker 上判断事务消息会用到 sysFlag。

- 设置好系统标记之后，生产者就像发送普通消息一样调用 NettyRemotingClient 将消息发送给 Broker，Broker 收到请求，根据RequestCode.SEND_MESSAGE 调用 SendMessageProcessor.sendMessage 处理请求。

- Broker 获取消息属性中 TRAN_MSG 属性，Flase 就走普通消息存储流程。如果为 True 则调用 TransactionalMessageService.prepareMessage 存入 Half 消息。

  > - 将原消息Topic 及 queueId 存入消息属性。Topic 替换为 RMQ_SYS_TRANS_HALF_TOPIC，queueId = 0
  >
  > - 然后调用 DefaultMessageStore.putMessage 按普通消息的步骤存入prepare消息
  >
  > - RocketMQ 会开启一个定时任务从 Topic 为 HALF_TOPIC 中拉取消息进行消费，根据生产者组获取一个服务提供者发送回查事务状态请求，根据事务状态来决定是提交或回滚消息。

- Half 消息存入之后，Broker 将存储结果返回给 Producer，Producer 收到发送成功的结果就调用业务实现的TransactionListener.executeLocalTransaction 方法执行本地事务，并保存本地事务执行结果。

### 提交/回滚

- 本地事务执行完成之后调用 DefaultMQProducerImpl.endTransaction 处理事务执行结果，其实就是 2PC 中的提交/回滚阶段。根据本地事务执行结果，向 Broker 发送 RequestCode.END_TRANSACTION 请求，告诉 Broker 是提交还是回滚。

- Broker 收到请求，根据RequestCode.END_TRANSACTION找到 EndTransactionProcessor 处理事务结果。根据事务的不同状态做不同的处理。

- 如果事务状态为TRANSACTION_NOT_TYPE (UNKOWN) 直接返回 null，不做处理。

  > 本地事务执行需要一定时间，Producer 在收到 half 消息存储结果时，本地事务状态通常都是 UNKOWN 的，也就是生产者此时发送的 RequestCode.END_TRANSACTION 请求不会做任何处理。那真正的提交在什么时候呢？

- 事务状态为 TRANSACTION_COMMIT_TYPE 则进行事务提交，简单来说事务提交就是恢复消息原本的主题，并存入 CommitLog 及 ConsumerQueue，这样 Consumer 就可以正常消费。同时还需要将消息添加到 RMQ_SYS_TRANS_OP_HALF_TOPIC (下文简称 OP_TOPIC ) 中，标记当前的事务消息已经提交。

  这里我们涉及了 3 个 Topic：在预提交阶段，我们将 Half 消息存入 HALF_TOPIC。在提交阶段我们有恢复了消息的原 Topic，并且在 OP_TOPIC 添加了一条消息，用于标记当前的事务消息已经提交。也就是如果消息在 OP_TOPIC 中存在就表名已经提交或回滚。

  > - commitMessage：根据 offset 查询 Half 消息
  > - 从消息属性中恢复消息主题、消费队列，构建新的消息对象
  >
  > - sendFinalMessage：将第二步构建的消息存储入commitlog中，因为恢复了消息主题等参数，所以可以被消费者消费。
  >
  > - deletePrepareMessage：将 Half 消息存储到 RMQ_SYS_TRANS_OP_HALF_TOPIC 主题中，表示该事务消息已经处理过（提交或回滚）

- 事务状态为 TRANSACTION_ROLLBACK_TYPE 则进行回滚，回滚与提交类似，只是回滚无须恢复消息原主题，直接将消息存储在 OP_TOPIC 主题中，表示已处理过该消息。

### 回查

本地事务执行完了需要将执行结果状态发送给 Broker，但是由于网络 NPC 问题，我们无法保证请求定会被正常接收。如果发生了网络错误，导致没有正常提交或者回滚要怎么处理呢？很多时候我们并不是怕发生问题，而且事实上问题总会发生，关键在于问题发生后我们采取什么样的措施。RocketMQ 解决这个问题的措施是增加补偿机制：定时回查本地事务状态，决定是提交还是回滚。

- 事务消息定时回查服务 TransactionalMessageCheckService 循环线程，默认每隔 60s 执行一次 TransactionalMessageCheckService.onWaitEnd，调用 TransactionalMessageService.check 执行回查。

- 获取所有 RMQ_SYS_TRANS_HALF_TOPIC 队列，即预提交消息队列

- 遍历预提交队列，每个队列处理 60s。

- 获取当前预提交队列对应的 RMQ_SYS_TRANS_OP_HALF_TOPIC，该主题队列用于记录已经提交或回滚的事务消息

- 获取 HALF、OP 队列当前进度

- 从 OP 队列拉取 32 条消息，因为 OP 消息记录事务已经被提交或回滚。

- 判断是否需要回查：主要根据消息有效时间、本地事务超时时间及消息是否在 OP 队列中 (OP队列中存在则已经提交或回滚，不需要回查)。如果无法判断是否回查，会从新拉取 32 条 OP 消息进行判断。

  > 判断是否需要回查的逻辑及较为复杂，包含多个很多参数，但是最重要的就是是否在 OP 队列中，如果对其他细节感兴趣可以到源码中详细看看，三此君给大家添加了详细的备注。

- 调用 TransactionalMessageCheckListener.resolveHalfMsg 异步发送回查消息

- 更新 HALF、OP 处理进度

> 值得注意的是，rocketmq 并不会无休止的的信息事务状态回查，默认回查 15 次，如果 15 次回查还是无法得知事务状态，rocketmq 默认回滚该消息。

### 远程事务

RocketMQ 的事务消息及时到这里就已经了解的差不多了，但是我看完官方文档，以及几本书上的内容后，总是觉得少了些什么？就这？怎么解决分布式事务问题的？回看前面的例子，例如生产者就是订单服务，现在下单成功了要发消息去库存服务减库存。

如何保证库存服务正常执行？只能基于 RocketMQ 的重试机制，默认重试 15 次后进入私信队列。通过重试尽可能保证库存服务执行成功。但是，万一就是库存本来就不够，库存扣减无法执行成功，但是订单服务又创建成功了怎么办？这就是基于 MQ 的分布式事务方法无能为力的地方，如果消费方执行出现问题，无法回滚生产者本第事务。

更有甚者 RocketMQ 出问题怎么办？我只能说凉拌炒鸡蛋。基于 MQ 的分布式事务消息，可以说是默认 MQ 本身是可靠的。

分布式事务各种解决方案有其自身的使用场景，没有一种方法能够完全解决分布式事务问题。

![](https://cdn.jsdelivr.net/gh/sancijun/images/pics/qrcode_banner.webp)

## 参考文献

- [RocketMQ 官方文档](https://github.com/apache/rocketmq/tree/master/docs/cn)
- [RocketMQ 源码](https://github.com/apache/rocketmq/tree/master)
- [如何选择分布式事务解决方案？](https://mp.weixin.qq.com/s/2AL3uJ5BG2X3Y2Vxg0XqnQ)

- 丁威, 周继锋. RocketMQ技术内幕：RocketMQ架构设计与实现原理. 机械工业出版社, 2019-01.

- 李伟. RocketMQ分布式消息中间件：核心原理与最佳实践. 电子工业出版社, 2020-08.
- 杨开元. RocketMQ实战与原理解析. 机械工业出版社, 2018-06.
