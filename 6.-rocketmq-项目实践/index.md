# RocketMQ 项目实践


## 前    言

大家好，我是三此君，一个在自我救赎之路上的非典型程序员。

“一张图”系列旨在通过“一张图”系统性的解析一个板块的知识点：

- 三此君向来不喜欢零零散散的知识点，通过一张图将零散的知识点连接起来，能够让我们对一个板块有更深入、更系统的理解。
- 同时本系列尽可能的精炼，希望能够让大家花 20% 的时间，快速理解这个板块下 80% 的内容。

本文是“一张图”系列的第一个板块：一张图解析 RocketMQ。

- 为了叙述的方便，绘图的时候将整个系列分为许多小的模块，讲解的时候也是按照模块循序渐进的。[一张图解析 RocketMQ 原图](https://sm.ms/image/vPtlGbaqHhJ9Tcg)
- 一张图解析 RocketMQ 是会深入到源码层面，但是文中不会粘贴源码。三此君在看源码的时候写了很多备注，可以降低大家看源码的难度，需要的同学自行到三此君的仓库中 Fork：[rocketmq release-4.3.0](https://github.com/sancijun/rocketmq/tree/release-4.3.0)

![一张图进阶 RocketMQ](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508151240.jpeg)

三此君经常会遇见这样的问题，理论学完觉得啥都会了，实践的时候又觉得啥都不会了。今天，我们通过一个真实的项目来看看 RocketMQ 在生产实践中如何使用，以及使用过程中会遇到什么问题，这些问题又如何解决？

消息队列在生产实践中是很常见的中间件，甚至很多团队不管三七二十一，适不适用都往消息队列里面扔。我们也无法穷尽消息队列的所有场景及问题，所以本文只是通过一个简单的项目，演示 RocketMQ 的应用及问题。

## 项目概述

今天分享的项目几乎是每个应用都会有的模块，每个人都用过，甚至会让有些人产生“恐惧”。这个项目就是消息中心，也就是给大家发消息的，如果还觉得不熟悉的话我提醒下你，你的应用是不是有个红点（有的还有数字），你是不是经常收到 Xxx 应用给你发送的短信/邮件？这些就是消息中心要做的事儿，给大家发消息。

消息中心的业务逻辑很简单，就是根据各个业务放的需求，按照模块，给用户发送消息。而消息可能是短信、邮件、站内信、推送通知（屏幕下滑出现的消息）、微信推送等各种渠道的消息，消息的类型也可以氛围营销、账号、验证码等。

首先是一些声明：本项目是`三此君`根据 `Java3y` 的 `austin` 项目修改而来，用于演示 RocketMQ 项目实践及问题，主要修改点如下：

> 1. kafka 替换为 RocketMQ。
> 2. 为了展示 RocketMQ 项目实践及问题，尽量减少依赖项，故去掉 Apollo/xxl-job/graylog 等依赖。
> 3. 简化部署，更改 docker 模块，只需要一个命令即可部署好所有依赖的中间件。
> 4. 简化部署，austin/austin-admin 合并。
> 5. 去掉原有的业务上的去重逻辑，新增 `通用幂等/重试` 实现。
> 6. 消费堆积处理

### 1、整体架构

我们先来看看消息通知的整体架构。这里只体现一些和消息中心自身强相关的组件或服务，要应用与实际生产环境中，肯定还有注册中心，监控告警，链路追踪等一系列微服务基础设施。

![整体架构](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508153823.png)

### 2、整体流程

业务方 `business service` 调用 austin 消息发送接口（指定消息模板 ID 及接受者等参数），austin 调用 RocketMQ Producer API 将消息发送到 RocketMQ Broker。消费者消费消息并生成消息发送 Task 放入线程池中。线程被调度执行，不同渠道的消息使用不同的 Handler，Handler 调用三方服务发送消息。

![整体流程](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508153850.png)

项目的具体实现细节三此君就不在此赘述了，大家可以获取源码，根据上面的流程图看就行。如果对实现有任何问题可以留言或者加三此君微信私聊。

## 项目部署

### 1、获取源码

- GitHub：https://github.com/sancijun/austin.git
- Gitee：

### 2、安装 Docker 及 docker-compose
- [Windows 安装 Docker](https://www.runoob.com/docker/windows-docker-install.html)
- [MacOS 安装 Docker](https://www.runoob.com/docker/macos-docker-install.html)

### 3、安装中间件
进入项目 docker 目录，执行 docker-compose up -d，等待镜像下载，容器启动。你获取不了解 Docker 及 docker-compose，但是并不影响，我们只是方便大家部署，部署好之正常使用就行。启动之后可在 docker dashboard 查看相关容器：
![](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508153925.png)

- mysql: 127.0.0.1:13306    username: root    password:（空） 
- redis: 127.0.0.1：:6379    username: （空）   password:（空）
- rocketmq-broker: 127.0.0.1:10909
- rocketmq-namesrv: 127.0.0.1:9876
- rocketmq-console: 127.0.0.1:8081 （访问 127.0.0.1:8081 即可查看 RocketMQ 控制台）

### 4、启动 austin-web
修改 austin-web/src/main/resources/application.properties 配置，打包启动即可。

需要特别说明的是，SMS、WeChat 渠道的接入会比较麻烦，只是为了加深对 RocketMQ 的理解的话，可以只接入 Email。接入 Email 的方式很简单，以 QQ 邮箱为例：进入 QQ 邮箱->点击设置->账号->开启 POP3/SMTP 服务->生成 Token；

替换 application.properties 中邮箱相关配置 `account.emailAccount`

```properties
account.emailAccount = [{"email_10":{"host":"smtp.qq.com","port":465,"user":"your_email","pass":"your_token","from":"your_email"}}]
```

![image-20220418224003650](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508153945.png)

### 5、启动 austin-admin
进入 austin-admin 目录，执行以下命令：
```bash
# 安装依赖
npm i
# 打开服务
npm start
```
访问 localhost:3000
![austin-admin](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508153955.png)

### 6、验证

可以直接在 austin-admin 上点击测试，也可以使用 postman 调用接口，查看消息是否正常发送。

到这里，我们大致了解了 austin 项目，也能够正常的收到消息了。一切都是那么顺利，但是直觉告诉三此君，《没那么简单》。我发送的消息要是没送到怎么办？是因为生产者的问题还是

## 通用幂等

### 1、存在问题

网络二将军问题的存在使得消息的发送者往往要重复发送消息，直到收到接收者的确认才认为发送成功，但这往往又会导致消息的重复发送。由于网络二将军问题的存在，RocketMQ 需要通过重试保证消息的可靠性，也因此 RocketMQ 无法避免消息重复（Exactly-Once）。消息中心是重度依赖 RocketMQ 的能力，可能出现重复发送消息等问题，因此需要实现消费幂等。

### 2、MySQL 排重表

**业务唯一标识**：可以借助关系数据库进行去重。首先需要确定消息的唯一键，根据消息中心目前实现没有合适的字段可以作为唯一标识。

1.部分接口有requestMsgId字段，是上游业务传过来的随机数，当前用于接口层面的防重。但是requestMsgId本身跟业务唯一性没有关系，并且RocketMQ消费者测试没有针对requestMsgId做处理。

2.每条消息会有 messageId，当前的实现方式是 UUID。简单的改造 messageId = SHA256(message content+telephone/uid/email/fcmToken+timestamp)，在同一时间，同一用户，相同的内容应该不会重复发送（业务唯一）。以messageId为唯一key实现排重。

3.message

```sql
# 1.开始事务
begin; 
# 2.插入消息表（处理好主键冲突的问题）
insert into msg_center_tab_xxx values (messageId, ...)
# 3.提交事务
commit;
```

实现问题：

1.消息的消费逻辑必须是依赖于关系型数据库事务。如果消费的消费过程中还涉及其他数据的修改，例如Redis这种不支持事务特性的数据源，则这些数据是不可回滚的。

2.不是通用方案：如果其他模块有需要处理幂等的地方，每个模块需要单独处理。

### 3、Redis 记录消息状态

![Redis 通用幂等流程](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508154752.png)

1）以业务唯一标识 messageId 为 Key 从 Redis 获取消息记录状态，如果在是消费中则延迟消费，消费成功则直接返回成功。

2）如果消息记录不存在，则先将消息记录到Redis, 业务唯一标识为key，消息状态为消费中，需要加入合理的过期时间；

3）插入消息记录成功后执行原有的业务逻辑，执行失败的话删除消息表记录；

### 4、幂等实现

**幂等注解**

```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Idempotent {
    /** 幂等 Redis  Key 前缀，用于区分业务*/
    String prefix() default "";
    /** 组成幂等 Key 的参数，通过拼接前缀及参数生成幂等业务唯一key*/
    String[] params() default {};
  	/** 如果接口有多个参数，需要指定subKeys包含在哪个目标参数中 */
    String target() default "";
}
```

**幂等切面**

```java
@Around("@annotation(com.shopee.banking.annotation.Idempotent)")
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
  try {
    Integer status = 0;
    // 1.获取参数生成业务唯一标识 key，从 Redis 获取消息记录状态
    String key = getUniqueKey(joinPoint);
    // 2.插入消费状态记录
    boolean first = redisTemplate.opsForValue().setIfAbsent(key,status,60*60, TimeUnit.SECONDS);
    // 3.插入不成功：消费成功则直接返回，消费不成功则抛出异常；
    // 4.执行业务代码
    Object result = joinPoint.proceed();
    // 5.设置为消费成功
    redisTemplate.opsForValue().set(key, 1);
    return result;
  } catch (Throwable throwable) {
    // 6.消费失败删除消息状态记录，等待重试
    String key = getUniqueKey(joinPoint);
    redisTemplate.delete(key);
    throw throwable;
  }
}
```

## 消息堆积

### 1、背景

“2021-09-29 20:12，local反馈用户没有收到 Email OTP，通过监控面板发现，kafka消息堆积了约32w的消息”。原因是批量发送了大量消息，并且Email OTP 这样高优先级的消息和批量的不重要消息没有做区分。消息堆积后，重要的消息也无法正常发送。

这个问题 ID 市场已经临时处理。短期方案通过增加 kafka 分区数，增加消费者线程数处理堆积消息；长期方案将消息模板按优先级区分，高优先级和低优先级的消息通知通过topic隔离，互不影响。

目前 ID 市场就 2021-09-29 的消息堆积问题解决方案已经较为完善，但是 PH 市场 kafka 切换 RocketMQ，消息队列本身的特性不一致，如果再遇到消息堆积问题又如何处理呢？针对消息堆积还有没有其他的问题呢？同时本次事故除了消息堆积，还有个很重要的关注点是普通消息堆积，影响了重要的消息发送。

### 2、消息堆积原因

消息处理流程中，如果客户端的消费速度跟不上服务端的发送速度，未处理的消息会越来越多，这部分消息就被称为堆积消息。RocketMQ DefaultPushConsumer 通过长轮询从 Broker 拉取消息，缓存到本地，再提交给业务线程。

Broker 消息堆积：主要关注下 Broker 端消息堆积的情况。Broker 端消息堆积是由于消息的生产和消费速度不匹配了，通常是由于客户端的消费能力不足。而客户端消费能力不足通常是由于受 **消费耗时** 和 **消费并行度** 影响。想要避免和解决消息堆积问题，必须合理的控制消费耗时和消息并发度，其中消费耗时的优先级高于消费并发度，必须先保证消费耗时的合理性，再考虑消费并发度问题。

客户端消息堆积：客户端堆积我们不经常遇到，[RocketMQ 的消息堆积](https://www.jianshu.com/p/58bb1610fb8c) 这篇文档记录了客户端消息堆积的内容。简单来说就是，存在1 个消费者实例负责消费 n 个 ConsumeQueue，Push 消费模式消是 RocketMQ 将消息拉倒本地放在缓存里（每次消息量大于100M会执行流控）。那么，n 个 线程同时拉取 100M 的消息缓存到本地，本地的消息就会堆积了。

### 3、消息堆积解决方案

#### 避免消息堆积和延迟（事前）

> 为了避免在业务使用时出现非预期的消息堆积和延迟问题，您需要在前期设计阶段对整个业务逻辑进行完善的排查和梳理。整理出正常业务运行场景下的性能基线，才能在故障场景下迅速定位到阻塞点。其中最重要的就是梳理消息的消费耗时和消息消费的并发度。
>
> - 梳理消息的消费耗时
>
>   通过压测获取消息的消费耗时，并对耗时较高的操作的代码逻辑进行分析。查询消费耗时，请参见[获取消息消费耗时](https://help.aliyun.com/document_detail/193952.htm#step-zbp-czw-m7t)。梳理消息的消费耗时需要关注以下信息：
>
>   - 消息消费逻辑的计算复杂度是否过高，代码是否存在无限循环和递归等缺陷。
>   - 消息消费逻辑中的I/O操作（如：外部调用、读写存储等）是否是必须的，能否用本地缓存等方案规避。
>   - 消费逻辑中的复杂耗时的操作是否可以做异步化处理，如果可以是否会造成逻辑错乱（消费完成但异步操作未完成）。
>
> - 设置消息的消费并发度
>
>   - 逐步调大线程的单个节点的线程数，并观测节点的系统指标，得到单个节点最优的消费线程数和消息吞吐量。
>   - 得到单个节点的最优线程数和消息吞吐量后，根据上下游链路的流量峰值计算出需要设置的节点数，节点数=流量峰值/单线程消息吞吐量。

#### 队列及消费者实例扩容（事中）

 RocketMQ 可以扩容 ComsumeQueue 及增加消费者线程数。

![image-20211029115145268](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508154850.png)

但是只是扩容 ConsumeQueue 及线程数对于已经堆积的消息是没有用的，因为已经堆积的消息本身还是存储在原来的 ConsumeQueue 中。而根据 RocketMQ 消费者负载均衡策略（默认平均分配），同一个 ConsumeQueue 中的消息只会被同一个消费者组中的一个实例消费。例如：原本有4个 ConsumeQueue，扩容到 16 个ConsumeQueue，根据负载策略。原本的消息所在的 4 个ConsumeQueue 依然最多只能被 4 个消费者实例消费。短时间内堆积的消息还是无法。针对以上问题，可以考虑如下方案：

方案一：如果消息是可以被丢弃的，在 rocketmq-comsole 可以重置消费位移。

方案二：首先我们要评估在 Topic 创建的时候就设置足够的 ConsumeQueue。那么 Topic 中 MessageQueue 的数量大于 Consumer 的实例数量，可以将 Consumer 扩容，MessageQueue 会进行 Rebalance 重新分配给 Consumer 实例，此时多个 Consumer 实例可以迅速消费掉堆积的消息，但是要考虑到的后续如果业务是否能够支撑突增的并发量。

方案三：出现消息大量堆积，并且 Topic 中 MessageQueue 的数量小于 Consumer 的实例数量，也就是上面描述的问题，仅仅增加 Consumer 是无效的。可以执行步骤：

1. 消费者入口增加 comsume.accumulation.exception.swtch，默认为 false。当开关开启时，不执行具体的消费逻辑，直接将消息发回到原有的 topic。
2. 当遇到消息严重堆积时，先执行 ConsumeQueue 动态扩容，接着执行 Consume 扩容，增加消费线程数，然后打开 comsume.accumulation.exception.swtch 开关时消息平均分配到扩容后的 ConsumeQueue 中。
3. 当存量的消息均匀分布在扩容后的 ConsumeQueue 时关闭开关，扩容后的 Consumer 根据负载策略可以较好的消费存量消息了。

#### 线上排查消费耗时（事中）

有很多可以参考的资料，这里就不复制粘贴了。

如何处理消息堆积：https://help.aliyun.com/document_detail/193952.htm?spm=a2c4g.11186623.0.0.52cc466csEvAYq#trouble-2004065

Arthas 用户文档：https://arthas.aliyun.com/doc/

综上，消息堆积通常由于消费者耗时和消费者并行度问题。为了避免消息堆积，在编码的时候应该详细评估消费者代码性能瓶颈，优化单此消费本身耗时。如果出现消息堆积大量堆积问题，根据情况不同有三种了应急策略：直接丢弃堆积消息，扩容消费者实例，ConsumeQueue 和 Consumer 同时扩容，并且将堆积的消息平均分布到扩容后的 ConsumeQueue。在应急之后还是应该分析耗时原因，具体是代码本身耗时，关联耗时？落到根本还是要解决消费者耗时问题上。

## 消息堆积

**1. 提高消费并行度**

绝大部分消息消费行为都属于 IO 密集型，即可能是操作数据库，或者调用 RPC，这类消费行为的消费速度在于后端数据库或者外系统的吞吐量，通过增加消费并行度，可以提高总的消费吞吐量，但是并行度增加到一定程度，反而会下降。所以，应用必须要设置合理的并行度。 如下有几种修改消费并行度的方法：

- 同一个 ConsumerGroup 下，通过增加 Consumer 实例数量来提高并行度（需要注意的是超过订阅队列数的 Consumer 实例无效）。可以通过加机器，或者在已有机器启动多个进程的方式。
- 提高单个 Consumer 的消费并行线程，通过修改参数 consumeThreadMin、consumeThreadMax 实现。

**2. 批量方式消费**

某些业务流程如果支持批量方式消费，则可以很大程度上提高消费吞吐量，例如订单扣款类应用，一次处理一个订单耗时 1 s，一次处理 10 个订单可能也只耗时 2 s，这样即可大幅度提高消费的吞吐量，通过设置 consumer 的 consumeMessageBatchMaxSize 返个参数，默认是 1，即一次只消费一条消息，例如设置为 N，那么每次消费的消息数小于等于 N。

**3. 跳过非重要消息**

发生消息堆积时，如果消费速度一直追不上发送速度，如果业务对数据要求不高的话，可以选择丢弃不重要的消息。例如，当某个队列的消息数堆积到 100000 条以上，则尝试丢弃部分或全部消息，这样就可以快速追上发送消息的速度。示例代码如下：

```java
    public ConsumeConcurrentlyStatus consumeMessage(
            List<MessageExt> msgs,
            ConsumeConcurrentlyContext context) {
        long offset = msgs.get(0).getQueueOffset();
        String maxOffset =
                msgs.get(0).getProperty(Message.PROPERTY_MAX_OFFSET);
        long diff = Long.parseLong(maxOffset) - offset;
        if (diff > 100000) {
            // TODO 消息堆积情况的特殊处理
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        }
        // TODO 正常消费过程
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    }    
```

**4. 优化每条消息消费过程**     

举例如下，某条消息的消费过程如下：

- 根据消息从 DB 查询【数据 1】
- 根据消息从 DB 查询【数据 2】
- 复杂的业务计算
- 向 DB 插入【数据 3】
- 向 DB 插入【数据 4】

这条消息的消费过程中有 4 次与 DB 的 交互，如果按照每次 5ms 计算，那么总共耗时 20ms，假设业务计算耗时 5ms，那么总过耗时 25ms，所以如果能把 4 次 DB 交互优化为 2 次，那么总耗时就可以优化到 15ms，即总体性能提高了 40%。所以应用如果对时延敏感的话，可以把 DB 部署在 SSD 硬盘，相比于 SCSI 磁盘，前者的 RT 会小很多。

![](https://cdn.jsdelivr.net/gh/sancijun/images/pics/qrcode_banner.webp)

## 参考文献

- [RocketMQ 官方文档](https://github.com/apache/rocketmq/tree/master/docs/cn)

- 丁威, 周继锋. RocketMQ技术内幕：RocketMQ架构设计与实现原理. 机械工业出版社, 2019-01.

- 李伟. RocketMQ分布式消息中间件：核心原理与最佳实践. 电子工业出版社, 2020-08.
- 杨开元. RocketMQ实战与原理解析. 机械工业出版社, 2018-06.
