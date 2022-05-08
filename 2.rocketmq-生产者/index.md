# RocketMQ 生产者


## 前    言

大家好，我是三此君，一个在自我救赎之路上的非典型程序员。

“一张图”系列旨在通过“一张图”系统性的解析一个板块的知识点：

- 三此君向来不喜欢零零散散的知识点，通过一张图将零散的知识点连接起来，能够让我们对一个板块有更深入、更系统的理解。
- 同时本系列尽可能的精炼，希望能够让大家花 20%的时间，快速理解这个板块下 80% 的内容。

本文是“一张图”系列的第一个板块：一张图解析 RocketMQ。

- 为了叙述的方便，绘图的时候将整个系列分为许多小的模块，讲解的时候也是按照模块循序渐进的。[一张图解析 RocketMQ 原图](https://sm.ms/image/vPtlGbaqHhJ9Tcg)
- 一张图解析 RocketMQ 是会深入到源码层面，但是文中不会粘贴源码。三此君在看源码的时候写了很多备注，可以降低大家看源码的难度，需要的同学自行到三此君的仓库中 Fork：[rocketmq release-4.3.0](https://github.com/sancijun/rocketmq/tree/release-4.3.0)

![一张图进阶 RocketMQ](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508151240.jpeg)

本文是“一张图解析 RocketMQ”第2 篇，对 RocketMQ 不了解的同学可以先看看三此君的 [RocketMQ 概述]()。在了解了 RocketMQ 的整体架构之后，我们来深入的分析下生产者消息发送的设计与实现。本文从一个生产者示例开始，以两行代码为切入点，逐步剖析生产者启动流程、同步消息发送流程、RocketMQ 的通信机制，最后简单过一下异步消息发送流程。

## 生产者示例

消息分为同步消息、异步消息和单向消息，简单来说：

- 同步消息：消息发送之后会等待 Broker 响应，并把响应结果传递给业务线程，整个过程业务线程在等待。
- 异步消息：调用异步发送 API，Producer 把消息发送请求放进线程池就返回。逻辑处理，网络请求都在线程池中进行，等结果处理好之后回调业务定义好的回调函数。
- 单向消息：只负责发送消息，不管发送结果。

我们先来回顾下同步消息发送的例子：

```java
public class SyncProducer {
    public static void main(String[] args) throws Exception {
        // 实例化消息生产者Producer
        DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");
        // 设置NameServer的地址
        producer.setNamesrvAddr("localhost:9876");
        // 启动Producer实例
        producer.start();
        // 创建消息，并指定Topic，Tag和消息体
        Message msg = new Message("Topic1","Tag", "Key", "Hello world".getBytes("UTF-8")); 
        // 发送消息到一个Broker
        SendResult sendResult = producer.send(msg);
      	// 通过sendResult返回消息是否成功送达
        System.out.printf("%s%n", sendResult);
        // 如果不再发送消息，关闭Producer实例。
        producer.shutdown();
    }
}
```

- 首先，实例化一个生产者 `producer`，并告诉它 NameServer 的地址，这样生产者才能从 NameServer 获取路由信息。
- 然后 `producer` 得做一些初始化（这是很关键的步骤），它要和 NameServer 通信，要先初始化通信模块等。
- `producer` 已经准备好了，那得准备好要发的内容，把 "Hello World" 发送到 Topic1。
- 内容准备好，那  `producer` 就可以把消息发送出去了。`producer` 怎么知道 Broker 地址呢？他会去 NameServer 获取路由信息，得到 Broker 的地址是 localhost:10909，然后通过网络通信将消息发送给 Broker。
- 生产者发送的消息通过网络传输给 Broker，Broker 需要对消息按照一定的结构进行存储。存储完成之后，把存储结果告知生产者。

其中有两个关键的地方：`producer.start()` 及 `producer.send()`，也就是生产者初始化及消息发送。我们以这两行代码为切入点，看看 RocketMQ Producer 的设计与实现。

> Tips：因为本文是RocketMQ 设计与实现分析，虽然不会粘贴任何源码，但是图文中会有大量的类名和方法名，看的时候不必执着于这些陌生的类名和方法名，三此君会解释这些类和方法的用途。

## 生产者启动

我们实例化一个生产者 DefaultMQProducer，并调用 DefaultMQProducer.start() 方法进行初始化：

![生产者启动流程](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508150321.png)

启动流程比较长，其实最重要的就是初始化了通信模块，并启动了多个定时任务，这些在后面的消息发送过程中都会用到：

- 检查配置是否合法：生产者组名是否为空、是否满足命名规则、长度是否满足等。
- **启动通信模块服务 Netty RemotingClient**：RemotingClient 是一个接口，底层使用的通讯框架是Netty，提供了实现类 NettyRemotingClient，RemotingClient 在初始化的时候实例化 Bootstrap，方便后续用来创建 SocketChannel；后文会介绍 RocketMQ 的通信机制，大家稍安勿躁。
- **启动 5 个后台定时任务**：定时更新 NameServerAddr 信息，**定时更新 topic 的路由信息**，定时向 Broker 发送心跳及清理下线的 Broker，定时持久化 Consumer 的 Offset 信息，定时调整线程池；
  生产者每 30s 会从某台 NameServer 获取 Topic 和 Broker 的映射关系（路由信息）存在本地内存中，如果发现新的 Broker 就会和其建立长连接，每 30s 会发送心跳至 Broker 维护连接。

> Tips：生产者为什么要启动消息拉取服务？重平衡服务是什么？简单来说，这两个服务都是用于消费者的，这里我们暂且不理会。消息拉取服务 pullMessageService 是从 Broker 拉取消息的服务 ，重平衡服务 rebalanceService 用于消费者的负载均衡，负责分配消费者可消费的消息队列。

## 同步发送

总体上讲，消息发送可以划分为三个层级：

- 业务层：准备需要发送的消息。
- 消息处理层：获取业务发送的 Message，经过一系列的参数检查、消息发送准备、参数包装等操作。
- 通信层：基于 Netty 封装的一个网络通信服务，将消息发送给 Broker。

![同步消息发送](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508150447.png)

我们通过前面的示例来看整个同步消息发送的处理流程，整个过程我们的主要目标就是把消息发送到 Broker：

- 第一步：业务层构建待发送消息  `Message msg = new Message("Topic1","Tag", "Key", "Hello world".getBytes("UTF-8"));`

- 第二步：然后我们调用 `producer.send(msg)` 发送消息，可是 producer 怎么知道发给谁呢？消息本身又需要经过哪些处理呢？我们进入调用链直到 **sendDefaultImpl**

  - 检查消息是否为空，消息的 Topic 的名字是否为空或者是否符合规范，消息体大小是否符合要求，最大值为4MB，可以通过 maxMessageSize 进行设置。

  - 执行 tryToFindTopicPublishInfo() 方法：获取 Topic 路由信息，如果不存在则抛出异常。如果本地缓存没有路由信息，就通过Namesrv 获取路由信息，更新到本地。消息构建的时候我们指定了消息所属 Topic，根据 Topic 路由信息我们可以找到对应的 Broker。

    > Tips：从 NameServer 获取的路由信息 TopicRouteData 会包含指定 Topic 的 topicQueueTable、brokerAddrTable，如果获取不到路由信息会使用默认的 topic 名称 "TBW102" 去获取路由信息。
    >
    > TBW102 就是接受自动创建主题的， Broker 启动会把这个默认主题登记到 NameServer，这样当 Producer 发送新 Topic 的消息时候就得知哪个 Broker 可以自动创建主题，然后发往那个 Broker。 Broker 接受到这个消息的时候发现没找到对应的主题，但是它接受创建新主题，这样就会创建对应的 Topic 路由信息。

  - 计算消息发送的重试次数，同步重试和异步重试的执行方式是不同的。在同步发送情况下如果发送失败会默认重投两次（默认retryTimesWhenSendFailed = 2），并且不会选择上次失败的 Broker，会向其他 Broker 投递。

  - 执行队列选择方法 selectOneMessageQueue()。根据 lastBrokerName（上次发送消息失败的 Broker 的名字）和 Topic 路由选一个 MessageQueue。
    首次发送 lastBrokerName 为 null，采用轮询策略选择一个 MessageQueue。如果上次发送失败，也是采用轮询策略选择一个 MessageQueue，但是会跳过属于上次发送失败 Broker 的 MessageQueue，也就是换一个 Broker 发送。

    > Tips：选择一个 MessageQueue，什么是 MessageQueue 呢？这和 Broker 的存储结构相关，我们会在存储部分详细介绍，这里先说结论，每个 Topic 默认会有 4 个 MessageQueue，每个 MessageQueue 有不同的 queueId(0-3)。 
    >
    > 我们也可以通过sendLatencyFaultEnable 来设置是否总是发送到延迟级别较低的Broker，默认值为False，我么这里就不展开讨论了。

  - 执行 sendKernelImpl() 方法。

- 第三步：sendDefaultImpl 做了一系列逻辑处理，我们已经得到了待发送的 BrokerName，而我们的目标是把消息发送到 Broker。sendKernelImpl 方法是发送消息的核心方法，主要用于准备通信层的入参（比如Broker地址、请求体等），将请求传递给通信层。

  - 根据 MessageQueue.brokerName 获取 Broker IP 地址，给message添加全局唯一 ID。

    > Tips：sendKernelImpl 也有很多的逻辑处理，我们暂时先略过这里的压缩、事务消息、钩子函数、重试消息：
    >
    > 对大于4k的普通消息进行压缩，并设置消息的系统标记为MessageSysFlag.COMPRESSED_FLAG。
    >
    > 如果是事务Prepared消息，则设置消息的系统标记为MessageSysFlag.TRANSACTION_PREPARED_TYPE
    >
    > 如果注册了消息发送钩子函数，则执行消息发送之前的增强逻辑，通过DefaultMQProducerImpl#registerSendMessageHook注册钩子处理类，并且可以注册多个。
    >
    > 构建发送消息请求头：生产者组、主题名称、默认创建主题Key、该主题在单个Broker默认队列数、队列ID（队列序号）、消息系统标记（MessageSysFlag）、消息发送时间、消息标记、消息扩展属性、消息重试次数、是否是批量消息等
    >
    > 处理重试消息。

  - 调用 MQClientAPIImpl.sendMessage()，首先构建一个远程请求 RemotingCommand，根据发送类型（同步或异步）调用不同的通信层实现方法。我们这里是同步消息，则调用 `RemotingClient.invokeSync()。`

  - 处理返回结果，将通信层返回的结果封装成 SendResult 对象返回给业务层。

- 第四步：RemotingClient 是基于 Netty 实现的，熟悉 Netty 的同学已经大概知道后面的流程，不熟系的同学也没有关系，这里先混个眼熟，下面我们会对 Netty 做简单的介绍。

  - RemotingClient.invokeSync() 先是通过 Broker Addr 获取或者创建 Netty Channel。先从 channelTables Map 本地缓存中，以Broker Addr 为 key 获取 Channel，没有获取到则通过 Netty Bootstrap.connect( Broker Addr) 创建 Channel，并放入缓存。
  - 然后生成<opaque, ResponseFuture>的键值对放入 responseTable 缓存中，结果返回的时候根据 opaque 从缓存中获取结果。
  - 调用 channel.writeAndFlush() 将消息通过网络传输给指定 Broker。这里是 Netty 框架的 API，已经不在 RocketMQ 范畴。
  - 调用 ResponseFuture.waitResponse() 方法，直到 Netty 接收 Broker的返回结果。其实就是执行 countDownLatch.await()。

- 第五步：结果处理及返回。

  - Broker 处理结果返回，Netty 产生可读事件，由 Channelhandler 处理可读事件，这里是 NettyClientHandler.channelRead0()接收写入数据，处理可读事件。
  - 然后处理返回结果，从  responseTable 取出 ResponseFuture，并执行 responseFuture.putResponse()。实际上就只执行 countDownLatch.countDown() 唤醒第四步中等待的调用线程，返回 Broker 的处理结果 RemotingCommand。
  - 结果层层返回，直到 MQClientAPIImpl.sendMessageSync() 出手了，这里调用 MQClientAPIImpl.processSendResponse() 处理返回结果，封装成 SendResult 对象返回给业务层。

到这里，生产者已经将消息发送到指定的 Broker 了，其中包括了消息的层层校验及封装；还有很重要的是如何选择一个 MessageQueue 进行发送（重试），重试是保证消息发送可靠的关键步骤；最后通过 Netty 将请求发送给 Broker。我们先不管 Broker 收到请求如何处理，但是要明白消息如何送到 Broker 进行存储，需要对 Netty 有简单的理解。

## 通信机制

### Netty 介绍

Netty 有很多概念，等介绍完概念大家都困了，我们就不过多介绍了，直接来看看 Netty 的基础流程，能够帮助我们更好的理解 RocketMQ 即可。

![netty 机制](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508150529.png)

1. Netty 服务端启动初始化两个线程组 **BossGroup & WorkerGroup**，分别用于处理**客户端连接及网络读写**。
2. Netty 客户端启动初始化一个线程组， 用于处理请求及返回结果。
3. 客户端 **connect** 到 Netty 服务端，创建用于 **传输数据的 Channel**。
4. Netty 服务端的 BossGroup 处理客户端的连接请求，然后把剩下的工作交给 WorkerGroup。
5. 连接建立好了，客户端就可以利用这个连接发送数据给 Netty 服务端。
6. Netty WorkerGroup 中的线程使用 **Pipeline(包含多个处理器 Handler)** 对数据进行处理。
7. Netty 服务端的处理完请求后，返回结果也经过 Pipeline 处理。
8. Netty 服务端通过 Channel 将数据返回给客户端。
9. 客户端通过 Channel 接收到数据，也经过 Pipeline 进行处理。

### Netty 示例

我们先用 Netty 实现一个简单的 服务端/客户端 通信示例，我们是这样使用的，那 RocketMQ 基于 Netty 的通信也应该是这样使用的，不过是在这个基础上封装了一层。主要关注以下几个点：服务端什么时候初始化的，服务端实现的 Handler 做了什么事？客户端什么时候初始化的，客户端实现的 Handler 做了什么事？

**Netty 服务端初始化**：初始化的代码很关键，我们要从源码上理解 RocketMQ 的通信机制，那肯定会看到类似的代码。根据上面的流程来看，首先是实例化 bossGroup 和 workerGroup，然后初始化 Channel，从代码可以看出我们是在 Pipeline 中添加了自己实现的 Handler，这个 Handler 就是业务自己的逻辑了，那 RocketMQ 要处理数据应该也需要实现相应的 Handler。

```java
public class MyServer {
    public static void main(String[] args) throws Exception {
        //创建两个线程组 boosGroup、workerGroup
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            //创建服务端的启动对象，设置参数
            ServerBootstrap bootstrap = new ServerBootstrap();
            //设置两个线程组boosGroup和workerGroup
            bootstrap.group(bossGroup, workerGroup)
                //设置服务端通道实现类型    
                .channel(NioServerSocketChannel.class)
                //使用匿名内部类的形式初始化Channel对象    
                .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            //给pipeline管道添加处理器
                            socketChannel.pipeline().addLast(new MyServerHandler());
                        }
                    });//给workerGroup的EventLoop对应的管道设置处理器
            //绑定端口号，启动服务端
            ChannelFuture channelFuture = bootstrap.bind(6666).sync();
            //对关闭通道进行监听
            channelFuture.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

**实现自定义的服务端处理器 Handler**：自定义的 Handler 需要实现 Netty 定义的 HandlerAdapter，当有可读事件时就会调用这里的 channelRead() 方法。等下我们看 RocketMQ 通信机制的时候留意RocketMQ 自定义了哪些 Handler，这些 Handler 有做了什么事。

```java
/**
 * 自定义的Handler需要继承Netty规定好的 HandlerAdapter 才能被Netty框架所关联，有点类似SpringMVC的适配器模式
 **/
public class MyServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        //获取客户端发送过来的消息
        ByteBuf byteBuf = (ByteBuf) msg;
        System.out.println("收到" + ctx.channel().remoteAddress() + "发送的消息：" + byteBuf.toString(CharsetUtil.UTF_8));
        //发送消息给客户端
        ctx.writeAndFlush(Unpooled.copiedBuffer("服务端已收到消息，记得关注三此君，记得三连", CharsetUtil.UTF_8));
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //发生异常，关闭通道
        ctx.close();
    }
}
```

**Netty 客户端初始化**：Netty 客户端，在 RocketMQ 中对应了 Producer/Consumer。在 Producer 启动中有一步是启动通信模块服务，其实就是初始化 Netty 客户端。客户端也需要先实例化一个 NioEventLoopGroup，然后将自定义的 handler 添加到 Pipeline，还有很重要的一步是我们需要 connect 连接到 Netty 服务端。

```java
public class MyClient {

    public static void main(String[] args) throws Exception {
        NioEventLoopGroup eventExecutors = new NioEventLoopGroup();
        try {
            //创建bootstrap启动引导对象，配置参数
            Bootstrap bootstrap = new Bootstrap();
            //设置线程组
            bootstrap.group(eventExecutors)
                //设置客户端的Channel实现类型    
                .channel(NioSocketChannel.class)
                //使用匿名内部类初始化 Pipeline
                .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            //添加客户端Channel的处理器
                            ch.pipeline().addLast(new MyClientHandler());
                        }
                    })
            //connect连接服务端
            ChannelFuture channelFuture = bootstrap.connect("127.0.0.1", 6666).sync();
            //对Channel关闭进行监听
            channelFuture.channel().closeFuture().sync();
        } finally {
            //关闭线程组
            eventExecutors.shutdownGracefully();
        }
    }
}
```

**实现自定义的客户端处理器 Handler**：客户端处理器也继承自 Netty 定义的 HandlerAdapter，当 Channel 变得可读的时候（服务端数据返回）会调用我们自己实现的 channelRead()。

```java
public class MyClientHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        //发送消息到服务端
        ctx.writeAndFlush(Unpooled.copiedBuffer("三此君，我正在看 RocketMQ 生产者发送消息~", CharsetUtil.UTF_8));
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        //接收服务端发送过来的消息
        ByteBuf byteBuf = (ByteBuf) msg;
        System.out.println("收到三此君的消息，我一定会三连的" + ctx.channel().remoteAddress() + byteBuf.toString(CharsetUtil.UTF_8));
    }
}
```

### RocketMQ 通信流程

RocketMQ 通信模块基于 Netty 实现，总体代码量不多。主要是 NettyRemotingServer和NettyRemotingClient，分别对应通信的服务端和客户端。根据前面的 Netty 示例，我们要理解 RocketMQ 如何基于 Netty 通信，只需要知道 4 个地方：NettyRemotingServer 如何初始化，NettyRemotingClient 初始化，如何基于 NettyRemotingClient 发送消息，无论是客户端还是服务端收到数据后都需要 Handler 来处理。

![RocketMQ 通信流程](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508150609.png)

- Broker/NameServer 需要启动 Netty 服务端。Broker 我们后面会进一步分析，只需要知道 Broker 启动的时候会调用 NettyRemotingServer.start() 方法初始化 Netty 服务器。
  主要做了 4 件事：配置 BossGroup/WorkerGroup NioEventLoopGroup 线程组，配置 Channel，添加 NettyServerHandler，调用 serverBootstrap.bind() 监听端口等待客户端连接。
- Producer/Consumer 需要启动 Netty 客户端，在生产者启动流程中 MQClientInstantce 启动通信服务模块，其实就是调用NettyRemotingClient.start() 初始化 Netty 客户端。
  主要做了 3 件事：配置客户端 NioEventLoopGroup 线程组，配置 Channel，添加 NettyClientHandler。
- 客户端配置了 Channel，但是 Channel 还没有创建，因为 Channel 肯定要和具体的 Server IP Addr 关联。在同步消息发送流程中，调用 NettyRemoteClient.invokeSync()，从 channelTables 缓存中获取或者创建一个新的 Channel，其实就是调用 bootstrap.connect() 连接到 NettyServer，创建用于通信的 Channel。
- 有了 Channel 后，Producer 调用 Channel.writeAndFlush() 将数据发送给服务器。NettyRemotingServer WorkerGroup 处理可读事件，调用 NettyServerHandler 处理数据。
- NettyServerHandler 调用 processMessageReceived方法。processMessageReceived 方法做了什么呢？通过传入的请求码 RequestCode 区别不同的请求，不同的请求定义了不同的 Processor。例如，是生产者存入消息使用 SendMessageProcessor，查询消息使用 QueryMessageProcessor，拉取消息使用 PullMessageProcessor。这些 Processor 在服务端初始化的时候，以 RequestCode 为 Key 添加到 Processor 缓存中。processMessageReceived 就是根据 RequeseCode 获取不同的 Processor，处理完后把结果返回给 NettyRemotingClient。
- NettyRemotingClient 收到可读事件，调用 NettyClientHandler 处理返回结果。NettyClientHandler也调用processMessageReceived 处理返回结果。processMessageReceived 从以 opaque 为 key ResponseTables 缓存冲取出 ResponseFuture，将返回结果设置到 ResponseFuture。同步消息则执行 responseFuture.putResponse()，异步调用执行回调。

## 异步发送

除了同步消息发送，RocketMQ 还支持异步发送。我们只需要在前面是示例中稍作修改就会得到一个异步发送示例，最大的不同在于发送的时候传入 SendCallback 接收异步返回结果回调。

```java
public class AsyncProducer {
    public static void main(String[] args) throws Exception {
        // 实例化消息生产者Producer
        DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");
        // 设置NameServer的地址
        producer.setNamesrvAddr("localhost:9876");
        // 启动Producer实例
        producer.start();
        // 创建消息，并指定Topic，Tag和消息体
        Message msg = new Message("Topic1","Tag", "Key", "Hello world".getBytes("UTF-8")); 
        // SendCallback 接收异步返回结果的回调
        producer.send(msg, new SendCallback() {
            @Override
            public void onSuccess(SendResult sendResult) {
            System.out.printf("关注呀！！！%-10d OK %s %n", index,sendResult.getMsgId());
            }
            @Override
            public void onException(Throwable e) {
            System.out.printf("三连呀！！！%-10d Exception %s %n", index, e);
            e.printStackTrace();
            }
        });
        // 如果不再发送消息，关闭Producer实例。
        producer.shutdown();
    }
}
```

同步发送个异步发送主要的过程都是一样的，不同点在于同步消息调用 Netty Channel.writeAndFlush 之后是 waitResponse 等待 Broker 返回，而异步消息是调用预先定义好的回调函数。

![异步消息](https://cdn.jsdelivr.net/gh/sancijun/images/pics/20220508150644.png)

异步消息和同步消息整体差不多，可以说在基于 Netty 实现异步消息比同步消息还要简单一下，我们这里主要来看一些不同点：

- 调用 DefaultMQProducer 异步发送接口需要我们定义 SendCallback 回调函数，在执行成功或者执行失败后回调。
- DefaultMQProducerImp 中的 send 方法会将异步发送请求封装成 Runable 提交到线程池，然后业务线程就直接返回了。
- sendDefaultImpl 计算重试同步和异步消息有区别，异步消息在这里不会重试，而是在后面结果返回的时候通过递归重试。
- 跟着调用链到 sendMessageAsync 方法，需要注意的是这里构建了 InvokeCallback 实例，ResponseFuture 会持有该实例，Netty 结果返回后调用该实例的方法。
- 下面就是正常的 Netty 数据发送流程，直到 Broker 处理完请求，返回结果。NettyRemotingClient 处理可读事件，NettyClientHandler 处理返回结果，调用 ResponseFuture.executeInokeCallback，进而调用 InvokeCallback.operationComplete.
- 如果 Broker 返回结果是成功的，则封装返回结果 SendResult，并回调业务实现的 SendCallback.onSucess 方法，更新容错项。
- 如果 Broker 返回失败，或出现任何异常则执行重试，重试超过 retryTimesWhenSendFailed 次则回调业务定义的 SendCallback.onException方法。

## 总结

以上就是 RocketMQ 消息发送的主要内容，我们简单的总结下：

- 生产者启动：主要是调用 NettyRemotingClient.start() 初始化 Netty 客户端，并启动 5 个后台线程；
- 消息发送：业务层封装发送的消息，逻辑层进行层层校验及封装，轮询策略选择一个 MessageQueue 发送(重试)，通信层基于 Netty 将消息发送给 Broker。
- 通信机制：基于 Netty 实现，只需要留意 NettyRemotingServer/NettyRemotingClient 的初始化，并且在通道变得可读/可写时，会调用 NettyServerhandler/NettyClienthandler 进行处理。
- 同步异步：同步和异步消息大同小异，只是同步消息通过 Netty 发送请求后会执行 ResponseFuture.waitResponse() 阻塞等待，而异步消息发送请求后不会等待，请求返回后调用 SendCallback 回调。

![banner](https://cdn.jsdelivr.net/gh/sancijun/images/pics/qrcode_banner.webp)

## 参考文献

- [RocketMQ 官方文档](https://github.com/apache/rocketmq/tree/master/docs/cn)
- [RocketMQ 源码](https://github.com/apache/rocketmq/tree/master)

- 丁威, 周继锋. RocketMQ技术内幕：RocketMQ架构设计与实现原理. 机械工业出版社, 2019-01.

- 李伟. RocketMQ分布式消息中间件：核心原理与最佳实践. 电子工业出版社, 2020-08.
- 杨开元. RocketMQ实战与原理解析. 机械工业出版社, 2018-06.
