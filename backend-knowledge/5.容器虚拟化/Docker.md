# Docker

[toc]

## Docker简介

### 1、是什么

[Docker](www.docker.com) 是一个开源的应用容器引擎，使用go语言编写。是一种容器技术解决方案实现。

出现的原因

- 虚拟机占用资源特别大 启动慢
- 传统的交换模式：只给程序，不给环境
    - 开发和运维之间的鸿沟
    - 减少运维的工作量


### 2、设计理念
解决了什么问题：由于不同的机器有不同的操作系统，以及不同的库和组件，在将一个应用部署到多台机器上需要进行大量的环境配置操作。

Docker 主要解决环境配置问题，它是一种虚拟化技术，对进程进行隔离，被隔离的进程独立于宿主操作系统和其它隔离的进程。使用 Docker 可以不修改应用程序代码，不需要开发人员学习特定环境下的技术，就能够将现有的应用程序部署在其它机器上。

### 3、能干什么
docker与传统虚拟机之间的差异 虚拟机：虚拟了整套环境 资源占用多，启动慢 docker轻量：没有Hypervisor和操作系统，只有docker引擎
Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。


### 5、为什么Docker会比VM快

虚拟机也是一种虚拟化技术，它与 Docker 最大的区别在于它是通过模拟硬件，并在硬件上安装操作系统来实现。

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/be608a77-7b7f-4f8e-87cc-f2237270bf69.png)

启动速度

启动虚拟机需要先启动虚拟机的操作系统，再启动应用，这个过程非常慢；

而启动 Docker 相当于启动宿主操作系统上的一个进程。

占用资源

虚拟机是一个完整的操作系统，需要占用大量的磁盘、内存和 CPU 资源，一台机器只能开启几十个的虚拟机。

而 Docker 只是一个进程，只需要将应用以及相关的组件打包，在运行时占用很少的资源，一台机器可以开启成千上万个 Docker。

1. docker有着比虚拟机更少的抽象层。由亍docker不需要Hypervisor实现硬件资源虚拟化,运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上docker将会在效率上有明显优势。

2. docker利用的是宿主机的内核,而不需要Guest OS。因此,当新建一个容器时,docker不需要和虚拟机一样重新加载一个操作系统内核。仍而避免引寻、加载操作系统内核返个比较费时费资源的过程,当新建一个虚拟机时,虚拟机软件需要加载Guest OS,返个新建过程是分钟级别的。而docker由于直接利用宿主机的操作系统,则省略了返个过程,因此新建一个docker容器只需要几秒钟。

### 6、优势
除了启动速度快以及占用资源少之外，Docker 具有以下优势：

**1.更容易迁移**

提供一致性的运行环境。已经打包好的应用可以在不同的机器上进行迁移，而不用担心环境变化导致无法运行。

**2.更容易维护**

使用分层技术和镜像，使得应用可以更容易复用重复的部分。复用程度越高，维护工作也越容易。

**3.更容易扩展**

可以使用基础镜像进一步扩展得到新的镜像，并且官方和开源社区提供了大量的镜像，通过扩展这些镜像可以非常容易得到我们想要的镜像。

### 7、使用场景

**1. 持续集成**

持续集成指的是频繁地将代码集成到主干上，这样能够更快地发现错误。

Docker 具有轻量级以及隔离性的特点，在将代码集成到一个 Docker 中不会对其它 Docker 产生影响。

**2. 提供可伸缩的云服务**

根据应用的负载情况，可以很容易地增加或者减少 Docker。

**3. 搭建微服务架构**

Docker 轻量级的特点使得它很适合用于部署、维护、组合微服务。


## 架构

Docker 使用客户端-服务器 (C/S) 架构模式，使用远程API来管理和创建Docker容器。

![img](http://cdn.processon.com/5f2531510791291b9975e44f?e=1596276577&token=trhI0BY8QfVrIGn9nENop6JAc6l5nZuxhjQ62UfM:X_kbt5XEyg6Cq5RGcXDIwoM67JU=)

### 1、镜像（Image）

Docker 镜像是用于创建 Docker 容器的模板

### 2、Docker Engine

**容器（Container）**
- Docker 容器通过 Docker 镜像来创建。容器与镜像的关系类似于面向对象编程中的对象与类。
- Docker 利用容器（Container）独立运行的一个或一组应用。
- 它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。
- 可以把容器看做是一个简易版的 Linux 环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。


**客户端 （Client）**
Docker 客户端通过命令行或者其他工具使用 Docker SDK 
(https://docs.docker.com/develop/sdk/) 与 Docker 的守护进程通信。

Docker Machine是一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure。

**服务器（Docker daemon）**
一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。


### 3、仓库 （Repository）

**Docker 仓库用来保存镜像**，可以理解为代码控制中的代码仓库。
Docker Hub(https://hub.docker.com) 提供了庞大的镜像集合供使用。

仓库注册服务器（Registry）

仓库(Repository)和仓库注册服务器（Registry）是有区别的，一个 Docker Registry 中可以包含多个仓库（Repository）。
每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。
通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。
我们可以通过 <仓库名>:<标签> 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。
Docker Hub也是一个仓库

运行过程 当client 执行命令docker run nginx时，client发送socket消息给docker守护进程， docker守护进程先在本地看下有没有这个镜像存在，如果不存在就去远程仓库下载，然后保存到本地； 然后通过 container run命令把这个镜像做成一个容器然后运行起来

hub.docker.com

docker 仓库地址 国内访问速度比较慢，可以使用阿里云加速 加速方式 

- 请求时制定镜像地址 docker pull registry.docker-cn.com/library/ubuntu:16.04 
- 启动docker守护进程时，添加--registry-mirror参数 
- 修改配置文件/etc/docker/daemon.json { "registry-mirrors": ["https://registry.docker-cn.com"] }

网易云加速（基本同上述阿里云）
配置 Json 串的地方不同了：{ "registry-mirrors": ["http://hub-mirror.c.163.com"]}

## 镜像

### 1、是什么

镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件。

文件和元数据的集合 + 镜像是分层的 + 不同的 image 可以共享相同的层 + 镜像本身是只读的


### 2、UnionFS(联合文件系统)

UnionFS（联合文件系统）：Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录


### 3、镜像加载原理

docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统称为UnionFS。

bootfs(boot file system)主要包含bootloader和kernel, bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。

rootfs (root file system) ，在bootfs之上。包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。 
平时我们安装进虚拟机的CentOS都是好几个G，为什么docker这里才200M？？

对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供 rootfs 就行了。由此可见对于不同的linux发行版, bootfs基本是一致的, rootfs会有差别, 因此不同的发行版可以公用bootfs。


### 4、分层镜像 

以我们的pull为例，在下载的过程中我们可以看到docker的镜像好像是在一层一层的在下载

为什么采用这种设计：最大的一个好处就是 - 共享资源

比如：有多个镜像都从相同的 base 镜像构建而来，那么宿主机只需在磁盘上保存一份base镜像，
同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。

### 5、特点
Docker镜像都是只读的
当容器启动时，一个新的可写层被加载到镜像的顶部。
这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。

### 6、更新镜像

docker commit 镜像提交
docker commit 提交容器副本使之成为一个新的镜像
docker commit -m=“提交的描述信息” -a=“作者” 容器ID 要创建的目标镜像名:[标签名]
docker commit --help docker commit -m="create image from current container" -a="panshen" 3a90f19f1669 "tomcat2:2.0"; 用已经存在的容器做一个新的镜像

例子
1. docker stop {container id} 停止容器运行
2. docker commit -m="{this update content}" -a="{author_id}" {container id} runoob/ubuntu:v2

### 7、构建镜像

1. 编写 Docker filed
2. docker build


## 数据卷

### 1、是什么

- Docker 将运用与运行的环境打包形成容器运行 ，运行可以伴随着容器，但是我们对数据的要求希望是持久化的。
- 容器之间希望有可能共享数据
- Docker容器产生的数据，如果不通过docker commit生成新的镜像，使得数据做为镜像的一部分保存下来，那么当容器删除后，数据自然也就没有了。
- 为了能保存数据在docker中我们使用卷。就是数据(一个文件或者文件夹)


### 2、能干什么

数据的永久化：完全独立于容器的生命周期，不会在容器删除时删除其挂载的数据卷，也不会存在类似的垃圾收集机制，对容器引用的数据卷进行处理
卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性：
卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷

特点：
1：数据卷可在容器之间共享或重用数据
2：卷中的更改可以直接生效
3：数据卷中的更改不会包含在镜像的更新中
4：数据卷的生命周期一直持续到没有容器使用它为止
日志系统存储（典型场景）

### 3、使用

**1. 命令添加**

docker run -it -v /宿主机绝对路径目录:/容器内目录 镜像名

-v 本机中的目录:容器中映射的目录

例子：docker run -v ~/local_dir:/data -it ubuntu /bin/bash

查看是否挂载成功：docker inspect 容器ID

命令(带权限)：docker run -it -v /宿主机绝对路径目录:/容器内目录:ro 镜像名

**2. DockerFile添加**

1. 根目录下新建mydocker文件夹并进入
2. 可在Dockerfile中使用VOLUME指令来给镜像添加一个或多个数据卷
VOLUME["/dataVolumeContainer","/dataVolumeContainer2","/dataVolumeContainer3"]  说明：  出于可移植和分享的考虑，用-v 主机目录:容器目录这种方法不能够直接在Dockerfile中实现。 由于宿主机目录是依赖于特定宿主机的，并不能够保证在所有的宿主机上都存在这样的特定目录。
3. DockerFile构建
#volume test FROM centos VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"] CMD echo "finished,--------success1" CMD /bin/bash
4. build后生成镜像
获得一个新镜像zzyy/centos
5. run容器
6. 通过上述步骤，容器内的卷目录地址已经知道对应的主机目录地址哪？？
7. 主机对应默认地址

注意：Docker挂载主机目录Docker访问出现cannot open directory .: Permission denied
解决办法：在挂载目录后多加一个--privileged=true参数即可


### 4、数据容器卷

从其他的容器中引用数据卷，命名的容器挂载数据卷，其它容器通过挂载这个(父容器)实现数据共享，挂载数据卷的容器，称之为数据卷容器

docker run -it --name n1 --volumes-from [container name] centos

**1. 总体介绍**

以上一步新建的镜像 zzyy/centos 为模板并运行容器dc01/dc02/dc03

它们已经具有容器卷
- /dataVolumeContainer1
- /dataVolumeContainer2

**2. 容器间传递共享**

- 先启动一个父容器dc01，在dataVolumeContainer2新增内容
- dc02/dc03继承自dc01
    - --volumes-from
    - docker run -it --name dc02 --volumes-from dc01 zzyy/centos，dc02/dc03分别在dataVolumeContainer2各自新增内容
- 回到dc01可以看到02/03各自添加的都能共享了
- 删除dc02后dc03可否访问，再进一步
- 新建dc04继承dc03后再删除dc03
- 结论：容器之间配置信息的传递，数据卷的生命周期一直持续到没有容器使用它为止

## Docker 容器互联

### 1、建立父子容器

1. 容器命名
当我们创建一个容器的时候，docker 会自动对它进行命名。另外，我们也可以使用 --name 标识来命名容器

2. 新建网络
docker network create -d bridge test-net
-d：参数指定 Docker 网络类型，有 bridge、overlay。
overlay 网络类型用于 Swarm mode

3. 连接容器
docker run -itd --name test1 --network test-net ubuntu /bin/bash
docker run -itd --name test2 --network test-net ubuntu /bin/bash

### 2、网络


**1. 单机**
Bridge Network
通过docker0网卡通信
Host Network
None Network

**2. 多机

Overlay Network
overlay 网络类型用于 Swarm mode


### 3、配置 DNS
1. 在宿主机的 /etc/docker/daemon.json 文件中增加以下内容来设置全部容器的 DNS
```json
{
    "dns" : [
        "114.114.114.114",
        "8.8.8.8"
    ]
}
```
2. 配置完，需要重启 docker 才能生效

3. 手动指定容器的配置

docker run -it --rm host_ubuntu --dns=114.114.114.114 --dns-search=test.com ubuntu

-h HOSTNAME 或者 --hostname=HOSTNAME： 设定容器的主机名，它会被写到容器内的 /etc/hostname 和 /etc/hosts。 

--dns=IP_ADDRESS： 添加 DNS 服务器到容器的 /etc/resolv.conf 中，让容器用这个服务器来解析所有不在 /etc/hosts 中的主机名。

--dns-search=DOMAIN： 设定容器的搜索域，当设定搜索域为 .example.com 时，在搜索一个名为 host 的主机时，DNS 不仅搜索 host，还会搜索 host.example.com。

## Dockerfile介绍

### 1、是什么

Dockerfile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本。

可以参考仓库中其他dockerfile文件的定义 https://github.com/docker-library/tomcat/blob/f58a6b4236cfe10672c9505aab5024100c9e084d/9.0/jre8/Dockerfile 

docker build [-f ...] 当前目录可以省略-f参数 docker build -t mytomcat .

### 2、基本规则
1：每条保留字指令都必须为大写字母且后面要跟随至少一个参数
2：指令按照从上到下，顺序执行
3：#表示注释
4：每条指令都会创建一个新的镜像层，并对镜像进行提交


### 3、Dockerfile执行流程
1. docker从基础镜像运行一个容器
2. 执行一条指令并对容器作出修改
3. 执行类似docker commit的操作提交一个新的镜像层
4. docker再基于刚提交的镜像运行一个新容器
5. 执行dockerfile中的下一条指令直到所有指令都执行完成

### 4、开发过程
1. 编写Dockerfile文件
2. docker build
3. docker run
- Dockerfile，需要定义一个Dockerfile，Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程(当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制)等等;
- Docker镜像，在用Dockerfile定义一个文件之后，docker build时会产生一个Docker镜像，当运行 Docker镜像时，会真正开始提供服务;
- Docker容器，容器是直接提供服务的。

### 5、指令

FROM：基础镜像，当前镜像是基于那个镜像

MAINTAINER：镜像维护者的姓名和邮箱地址


WORKDIR

容器创建后，默认在那个目录
指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。（WORKDIR 指定的工作目录，必须是提前创建好的）。

docker build 构建镜像过程中的，每一个 RUN 命令都是新建的一层。只有通过 WORKDIR 创建的目录才会一直存在。

EXPOSE：当前容器对外暴露的接口


ENV：用来构建镜像时设置环境变量

ENV MY_PATH /usr/mytest 这个环境变量可以在后续的任何RUN指令中使用，这就如同在命令前面指定了环境变量前缀一样； 也可以在其它指令中直接使用这些环境变量，  比如：WORKDIR $MY_PATH
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...

ADD：将宿主机目录下的文件copy到镜像且ADD命令会自动解压压缩包

ADD 不能加压zip包
ADD 的优点：在执行 <源文件> 为 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，会自动复制并解压到 <目标路径>。
ADD 的缺点：在不解压的前提下，无法复制 tar 压缩文件。会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。具体是否使用，可以根据是否需要自动解压来决定。


COPY：类似ADD，拷贝文件和目录到镜像中。
将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置
COPY [--chown=<user>:<group>] <源路径1>... <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]

<目标路径>：容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建。

VOLUME：容器数据卷，用来保存和持久化

VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
在启动容器 docker run 的时候，我们可以通过 -v 参数修改挂载点。


RUN：镜像构建时需要运行的命令


RUN：用于执行后面跟着的命令行命令。
RUN ["可执行文件", "参数1", "参数2"]
RUN 是在 docker build 运行

CMD：指定容器启动时需要运行的命令
多条CMD命令，只有最后一条生效
CMD命令会被docker run之后的参数替换
CMD 在docker run 时运行。

ENTRYPOINT：指定容器启动过程中需要运行的命令
把docker run命令的参数追加到后面
优点：在执行 docker run 的时候可以指定 ENTRYPOINT 运行所需的参数。
注意：如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效。
可以搭配 CMD 命令使用：一般是变参才会使用 CMD ，这里的 CMD 等于是在给 ENTRYPOINT 传参，以下示例会提到。


ARG构建参数，与 ENV 作用一至。
不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效，也就是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量。

构建命令 docker build 中可以用 --build-arg <参数名>=<值> 来覆盖。

ARG <参数名>[=<默认值>]


USER：用于指定执行后续命令的用户和用户组，这边只是切换后续命令执行的用户（用户和用户组必须提前已经存在）。
USER <用户名>[:<用户组>]


ONBUILD：当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发
用于延迟构建命令的执行。
Dockerfile 里用 ONBUILD 指定的命令，在本次构建镜像的过程中不会执行（假设镜像为 test-build）。当有新的 Dockerfile 使用了之前构建的镜像 FROM test-build ，这是执行新镜像的 Dockerfile 构建时候，会执行 test-build 的 Dockerfile 里的 ONBUILD 指定的命令。

HEALTHCHECK
用于指定某个程序或者指令来监控 docker 容器服务的运行状态。
HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

HEALTHCHECK [选项] CMD <命令> : 这边 CMD 后面跟随的命令使用，可以参考 CMD 的用法。

### 6、案例

错误
**注意：Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大。**
FROM centos
RUN yum install wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN tar -xvf redis.tar.gz

**以上执行会创建 3 层镜像。可简化为以下格式：**
FROM centos
RUN yum install wget \
&& wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
&& tar -xvf redis.tar.gz


Docker化一个Spring Boot应用
1. git pull openjdk


## 安装

Docker分为 CE（Community Edition: 社区版） 和 EE（Enterprise Edition: 企业版）

参考最权威的官方文档

安装社区版本就行 最权威的 centos 安装地址 https://docs.docker.com/engine/install/

CentOS：https://docs.docker.com/engine/install/centos/

Mac：https://docs.docker.com/docker-for-mac/install/

Windows：https://docs.docker.com/docker-for-windows/install/

配置仓库地址
1. 访问及注册阿里云，获取自己的专属加速地址 https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

2. 配置加速地址

不同环境不同

CentOs
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://xxx.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
Windows
针对安装了Docker for Windows的用户，您可以参考以下配置步骤：
在系统右下角托盘图标内右键菜单选择 Settings，打开配置窗口后左侧导航菜单选择 Docker Daemon。编辑窗口内的JSON串，填写下方加速器地址：
`{ "registry-mirrors": ["https://kw34q2d0.mirror.aliyuncs.com"] }`

Docker Desktop
Docker Desktop 是一个易于安装在你的Mac或者Windows上的，可以让你构建一个共享容器应用和微服务。
Docker Desktop包括Docker Engine，Docker CLi client，Docker Compose, Notary, Kubernetes, and Credential Helper.

## 命令

帮助命令
docker version
docker info  （docker相关的信息）
docker --help
man docker （docker 说明书）

docker command --help 更深入的了解指定的 Docker 命令使用方法。
例如我们要查看 docker stats 指令的具体使用方法：
docker stats --help

镜像命令
什么镜像 一个文件系统的某个目录 什么时容器 容器时镜像的实例，相当于镜像是一个类，容器是一个实例 ；每个镜像之间是隔离的 容器是动态的，镜像是静态的
```
docker images
docker images --help 可以查看
```

列出本地主机上的镜像，各个选项说明:
```
REPOSITORY：表示镜像的仓库源
TAG：镜像的标签
IMAGE ID：镜像ID
CREATED：镜像创建时间
SIZE：镜像大小
```
同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本，我们使用 REPOSITORY:TAG 来定义不同的镜像。
如果你不指定一个镜像的版本标签，例如你只使用 ubuntu，docker 将默认使用 ubuntu:latest 镜像
```
OPTIONS说明
-a 列出所有镜像
-q 只显示镜像ID
--digests：显示摘要信息
--no-trunc：不截断输出，显示完整的镜像ID
-f 根据提供的条件过滤输出
```
支持的过滤参数文档地址 https://docs.docker.com/engine/reference/commandline/images/

docker search [OPETIONS] 镜像名

starts 类似github上的stars official 是否官方 https://docs.docker.com/engine/reference/commandline/search/

网站 https://hub.docker.com

OPTIONS 说明
--no-trunc : 显示完整的镜像描述
-s : 列出收藏数不小于指定值的镜像。
--automated : 只列出 automated build类型的镜像；

docker pull 拉取镜像
docker pull 镜像名字[:TAG]

docker rmi 删除镜像
删除单个：
docker rmi 镜像id/镜像名称
docker rmi -f 镜像id/镜像名称
删除多个镜像： 
docker rmi id1 id2
docker rmi -f 镜像名1:TAG 镜像名2:TAG 
删除全部镜像： 
docker rmi ${docker images -qa}
docker rmi -f $(docker images -qa)

docker rmi `docker images -qa` $() ($+小括号)子shell命令

容器命令

\+ 创建容器 + 查看容器运行信息 + 启动容器 + 关闭容器


新建并运行（有镜像才能创建容器）


docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
- OPTIONS --name为容器指定新名称
          -d 后台运行
          -i交互方式运行
          -t伪终端
          -p端口映射
          -P随机端口映射


启动交互式容器
使用镜像centos:latest以交互模式启动一个容器,在容器内执行/bin/bash命令。
docker run -it centos /bin/bash 


列出所有运行的容器


docker ps [options]
-a :所有正在运行和运行过的
-l: 显示最近创建的容器
-n:显示最近创建的n个容器
-q:只显示容器id

![img](http://cdn.processon.com/5c360dbde4b056ae29e8d2fa?e=1547049933&token=trhI0BY8QfVrIGn9nENop6JAc6l5nZuxhjQ62UfM:UtIWuWE9JyKYVtcj4Pl5tbTjUJ4=)
docker ps status列中包含up，标识服务已经起来了


退出容器
exit / ctrl + d：退出并停止容器
ctrl+p+q:退出不停止容器


启动容器
docker start 容器id/名称

启动已经退出的容器


重启容器
docker restart 容器id/名称


停止容器
docker stop 容器id/名称


强制停止所有容器
docker kill 容器id/名称


删除容器
docker rm 容器id/名称


一次性删除多个容器
docker rm -f $(docker ps -a -q)
docker ps -a -q | xargs docker rm


以后台方式运行容器


docker run -d 容器
不占用当前终端

比如tomcat容器，有后台方式运行就不占用当前控制台
docker run -d tomcat
使用镜像centos:latest以后台模式启动一个容器
docker run -d centos
问题：然后docker ps -a 进行查看, 会发现容器已经退出
很重要的要说明的一点: Docker容器后台运行,就必须有一个前台进程.
容器运行的命令如果不是那些一直挂起的命令（比如运行top，tail），就是会自动退出的。
这个是docker的机制问题,比如你的web容器,我们以nginx为例，正常情况下,我们配置启动服务只需要启动响应的service即可。例如
service nginx start
但是,这样做,nginx为后台进程模式运行,就导致docker前台没有运行的应用,
这样的容器后台启动后,会立即自杀因为他觉得他没事可做了.
所以，最佳的解决方案是,将你要运行的程序以前台进程的形式运行


查看容器内运行的进程
docker top 容器ID


查看容器内部细节
docker inspect 容器ID


查看容器日志


docker logs -f -t --tail 容器ID
 docker run -d ubuntu /bin/sh -c "while true;do echo hello zzyy;sleep 2;done"
\*  -t 是加入时间戳
\*  -f 跟随最新的日志打印
\*  --tail 数字 显示最后多少条


进入容器
在使用 -d 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下指令进入


docker attach 容器ID
attach 直接进入容器启动命令的终端，不会启动新的进程


用法
\1. docker ps 获取container id
\2. docker attach {container id}
\3. exit


docker exec
推荐大家使用 docker exec 命令，因为此退出容器终端，不会导致容器的停止。
exec 是在容器中打开新的终端，并且可以启动新的进程
docker exec --help


用法
\1. docker ps 获取container id


\2. docker exec -it {container id} {command}
比如： docker exec -it 243c32535da7 /bin/bash
\3. exit


导出和导入容器


导出容器
如果要导出本地某个容器，可以使用 docker export 命令。
导出容器 1e560fca3906 快照到本地文件 ubuntu.tar：
docker export 1e560fca3906 > ubuntu.tar


导入容器快照
使用 docker import 从容器快照文件中导入为镜像
cat docker/ubuntu.tar | docker import - test/ubuntu:v1


通过指定 URL 或者某个目录来导入
docker import http://example.com/exampleimage.tgz example/imagerepo


容器 <->拷贝文件<->主机
docker cp 容器id/名称:容器中路径 主机路径
docker cp 主机路径 容器id/名称:容器中路径 

从容器中复制文件到主机 docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|- 从主机复制文件到容器 docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH


常用命令总结


引擎类型：针对引擎的命令
info    Display system-wide information        # 显示系统相关信息
images   List images                  # 列出系统当前镜像
ps     List containers                # 列出容器列表
export   Stream the contents of a container as a tar archive  # 导出容器的内容流作为一个 tar 归档文件[对应 import ]
import   Create a new filesystem image from the contents of a tarball # 从tar包中的内容创建一个新的文件系统映像[对应export]
events   Get real time events from the server      # 从 docker 服务获取容器实时事件


仓库
login   Register or Login to the docker registry server   # 注册或者登陆一个 docker 源服务器
logout   Log out from a Docker registry server      # 从当前 Docker registry 退出
search   Search for an image on the Docker Hub     # 在 docker hub 中搜索镜像
pull    Pull an image or a repository from the docker registry server  # 从docker镜像源服务器拉取指定镜像或者库镜像
push    Push an image or a repository to the docker registry server   # 推送指定镜像或者库镜像至docker源服务器


镜像类型：针对镜像的命令
search   Search for an image on the Docker Hub     # 在 docker hub 中搜索镜像
pull    Pull an image or a repository from the docker registry server  # 从docker镜像源服务器拉取指定镜像或者库镜像
push    Push an image or a repository to the docker registry server   # 推送指定镜像或者库镜像至docker源服务器
tag    Tag an image into a repository         # 给源中镜像打标签
rmi    Remove one or more images       # 移除一个或多个镜像[无容器使用该镜像才可删除，否则需删除相关容器才可继续或 -f 强制删除]
load    Load an image from a tar archive        # 从一个 tar 包中加载一个镜像[对应 save]
save    Save an image to a tar archive         # 保存一个镜像为一个 tar 包[对应 load]
build   Build an image from a Dockerfile        # 通过 Dockerfile 定制镜像
commit   Create a new image from a container changes  # 提交当前容器为新的镜像
history  Show the history of an image          # 展示一个镜像形成历史


容器类型：针对容器的命令
start   Start a stopped containers           # 启动容器
stop    Stop a running containers           # 停止容器
restart  Restart a running container          # 重启运行的容器
pause   Pause all processes within a container     # 暂停容器
unpause  Unpause a paused container           # 取消暂停容器
run    Run a command in a new container        # 创建一个新的容器并运行一个命令
kill    Kill a running container            # kill 指定 docker 容器
top    Lookup the running processes of a container  # 查看容器中运行的进程信息
logs    Fetch the logs of a container         # 输出当前容器日志信息
create   Create a new container             # 创建一个新的容器，同 run，但不启动容器
rm     Remove one or more containers         # 移除一个或者多个容器
diff    Inspect changes on a container's filesystem # 查看 docker 容器变化
attach   Attach to a running container         # 当前 shell 下 attach 连接指定运行镜像
exec    Run a command in an existing container     # 在已存在的容器上运行命令
wait    Block until a container stops, then print its exit code  # 截取容器停止时的退出状态值
inspect  Return low-level information on a container  # 查看容器详细信息
cp     Copy files/folders from the containers filesystem to the host path  #从容器中拷贝指定文件或者目录到宿主机中

docker port {container id / container name}
可以查看指定 （ID 或者名字）容器的某个确定端口映射到宿主机的端口号。

注意点：

docker进程使用Unix Socket而不是TCP端口。而默认情况下，Unix socket属于root用户，需要root权限才能访问。
运行docker命令最好使用root权限
docker守护进程启动的时候，会默认赋予名字为docker的用户组读写Unix socket的权限，因此只要创建docker用户组，并将当前用户加入到docker用户组中，那么当前用户就有权限访问Unix socket了，进而也就可以执行docker相关命令

sudo groupadd docker #添加docker用户组 sudo gpasswd -a $USER docker #将登陆用户加入到docker用户组中 newgrp docker #更新用户组 docker ps #测试docker命令是否可以使用sudo正常使用


