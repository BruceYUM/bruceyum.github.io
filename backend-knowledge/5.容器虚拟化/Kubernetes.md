# Kubernetes


[toc]

## 介绍说明

发展历史公
- 有云类型说明
- 资源管理器对比
- K8S 其优势


K8S 组件说明

- etcd
一个可信赖的分布式键值储存服务

- pod
Pod就是一个或多个Container的组合 
官网解释：A Pod (as in a pod of whales or pea pod) is a group of one or more containers (such as Docker containers),
 with shared storage/network, and a specification for how to run the containers.


- Pod的维护谁来做？——ReplicaSet
ReplicaSet通过selector来进行管理
官网解释：A ReplicaSet is defined with fields, including a selector that specifies how to identify Pods it can acquire, 
a number of replicas indicating how many Pods it should be maintaining, and a pod template specifying the data 
of new Pods it should create to meet the number of replicas criteria.

![img](http://cdn.processon.com/5f2e7ce60791297d38dd7471?e=1596885751&token=trhI0BY8QfVrIGn9nENop6JAc6l5nZuxhjQ62UfM:utplspyyi4ryfYkbYOzfNUafgZs=)

- Pod和ReplicaSet的状态如何维护和监测呢？—— Deployment 
Deployment 监测 Pod和ReplicaSet的变动
官网解释：A Deployment controller provides declarative updates for Pods and ReplicaSets.
You describe a desired state in a Deployment, and the Deployment controller changes the 
actual state to the desired state at a controlled rate. You can define Deployments to create
 new ReplicaSets, or to remove existing Deployments and adopt all their resources with new 
Deployments. 

![img](http://cdn.processon.com/5f2e7d937d9c083149a5dbb2?e=1596885923&token=trhI0BY8QfVrIGn9nENop6JAc6l5nZuxhjQ62UfM:R0gQ7kggytK5i72HJ-EP8pV1PSs=)

- Label
Labels是一个key-value键值对，可以给对象贴的标签，比如说给一组pod贴上这一类标签
官网解释：Labels are key/value pairs that are attached to objects, such as pods.
`YAML apiVersion: v1 kind: Pod metadata: name: nginx-pod labels: # 给对象打标签 app: nginx `

- Service
Service 可以这类Lobel的对象起名字，比如给一组Pod起名字
官网解释： An abstract way to expose an application running on a set of Pods as a network service.

With Kubernetes you don’t need to modify your application to use an unfamiliar service discovery
 mechanism. Kubernetes gives Pods their own IP addresses and a single DNS name for a set of Pods, 
and can load-balance across them. 

- Node
Pods运行在Node之上，Node表示的是一台机器，比如一台Centos虚拟机，一个Node可以包含多个services/pods
一个Node可以运行多个Pod
官网解释：A node is a worker machine in Kubernetes, previously known as a minion. 
A node may be a VM or physical machine, depending on the cluster. 
Each node contains the services necessary to run pods and is managed by the master components.

- 集群

多个Node共同组成集群

![img](http://cdn.processon.com/5f2e7f94f346fb718464da17?e=1596886437&token=trhI0BY8QfVrIGn9nENop6JAc6l5nZuxhjQ62UfM:E5wS7qCvibNBORXpUCPQViHM8nY=)

集群框架图

组件结构
![img](http://cdn.processon.com/5f2e77c1637689313ac34e27?e=1596884433&token=trhI0BY8QfVrIGn9nENop6JAc6l5nZuxhjQ62UfM:1vSeD2NSI-vUP8utYxU71wbuHmA=)

![img](http://cdn.processon.com/5f2e7830f346fb718464d313?e=1596884544&token=trhI0BY8QfVrIGn9nENop6JAc6l5nZuxhjQ62UfM:a5xVAEAyGXKz8UEt9AdQlRJv7vE=)


![img](http://cdn.processon.com/5f2e8b1f0791297d38dd7f69?e=1596889392&token=trhI0BY8QfVrIGn9nENop6JAc6l5nZuxhjQ62UfM:492JeFXTYEKKxOYT_VTYIxxwWDc=)

- kubectl
操作集群的客户端，也就是和集群打交道

**api server**
请求肯定是到达Master Node，然后再分配给Worker Node创建Pod之类的 关键是命令通过kubectl过来之后，要认证授权一下
请求过来之后，Master Node中谁来接收

**Scheduler**
API收到请求之后，接下来由Schedule协调调用哪个Worker Node创建Pod，Container之类的，有调度策略
https://kubernetes.io/docs/concepts/scheduling/kube-scheduler

**Controller Manager**
Scheduler通过不同的策略，真正要分发请求到不同的Worker Node上创建内容，具体由Controller Manager负责

**Worker Node的kubelet**
Worker Node接收到创建请求之后，具体Worker Node的Kubelet来负责

最终Kubelet会调用Docker Engine，创建对应的容器
在Node上需要有Docker Engine，不然怎么创建维护容器
会涉及到域名解析DNS的问题

**Dashboard **
监控面板，能够监测整个集群的状态

**ETCD**
集群中这些数据如何保存？分布式储存在ETCD之中
简化的架构图

![img](http://cdn.processon.com/5f2e8d121e085366ab169a90?e=1596889890&token=trhI0BY8QfVrIGn9nENop6JAc6l5nZuxhjQ62UfM:uWa-yNJPzJy6oG2OOOqiapg3fuQ=)

官网架构图

![img](http://cdn.processon.com/5f2e8d497d9c083149a5e8cb?e=1596889945&token=trhI0BY8QfVrIGn9nENop6JAc6l5nZuxhjQ62UfM:5zLGNnTBm6UrugXHs3SFoz2MPLg=)


- namespace

**资源隔离**
命名空间就是为了隔离不同的资源，比如：Pod、Service、Deployment等。
可以在输入命令的时候指定命名空间-n，如果不指定，则使用默认的命名空间：default。
 默认namespace为default
通过kubectl get namespaces查看


**selector**

选择器，筛选对象

```yaml 
apiVersion: apps/v1 
kind: Deployment 
metadata: 
name: nginx-deployment 
labels: 
app: nginx 
spec: 
replicas: 3 
selector: # 匹配具有同一个label属性的pod标签 
matchLabels: app: 
nginx template: # 定义pod的模板 
metadata: 
labels: 
app: nginx # 定义当前pod的label属性，app为key，value为nginx 
spec: 
containers: 
name: nginx image: 
nginx:1.7.9 ports: - containerPort: 80 
```

Label Selector
通过Label筛选/正则表达式的对象

一般Selector
通过selector的name匹配筛选对象


## kubernetes 安装

### 1、单机版
windows和mac可以通过 docker desktop安装

在线玩
在线play-with-k8s: https://labs.play-with-k8s.com/

Minikube
K8S单节点，适合在本地学习使用 
https://kubernetes.io/docs/setup/learning-environment/minikube/ 


### 2、构建k8s集群

linux安装
系统初始化
Kubeadm部署安装
常见问题分析

Cloud 上搭建
https://github.com/kubernetes/kops

CoreOs
https://coreos.com/tectonic/

kubeadm
本地多节点(这是官网推荐的搭建方式) 
https://github.com/kubernetes/kubeadm

优化方式二


### 3、感受一下kubernetes

查看连接信息
kubectl config view 
kubectl config get-contexts 
kubectl cluster-info


体验Pod
(1)创建pod_nginx.yaml|

```
apiVersion: v1 
kind: Pod 
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
   image: nginx
   ports:
   - containerPort: 8
```
(2)根据pod_nginx.yaml文件创建pod 
kubectl apply -f pod_nginx.yaml 

(3)查看pod 
kubectl get pods 
kubectl get pods -o wide 
kubectl describe pod nginx

(4)进入nginx容器 
\# kubectl进入 kubectl exec -it nginx bash # 通过docker进入 minikube ssh docker ps docker exec -it containerid bash

(5)访问nginx，端口转发
\# 若在minikube中，直接访问 # 若在物理主机上，要做端口转发 kubectl port-forward nginx 8080:80

(6)删除pod 
kubectl delete -f pod_nginx.yaml 


## 重要组件的概念

Pod

分类

自主式Pod（静态Pod）
静态Pod是由kubelet进行管理的，并且存在于特定的Node上。
不能通过API Server进行管理，无法与ReplicationController,Ddeployment或者DaemonSet进行关联，也无法进行健康检查。

控制器管理的Pod

控制器

ReplicationController
RC 一定要指出一次有多少个Pod副本在运行，RC可以控制一个或者一组Pods一直都是启动着并有效的。
官网解释：A ReplicationController ensures that a specified number of pod replicas are running at any one time. 
In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods 
is always up and available.
使用yaml文件的template字样
案例

```yaml apiVersion: v1 kind: ReplicationController # 表示要新建对象的类型 metadata: name: nginx spec: replicas: 3 # 表示受此RC管理的Pod需要运行的副本数 selector: # 表示需要管理的Pod的label，这里表示包含app: nginx的label的Pod都会被该RC管理 app: nginx template: # 表示用于定义Pod的模板，比如Pod名称、拥有的label以及Pod中运行的应用等通过改变RC里Pod模板中的镜像版本，可以实现Pod的升级功能 metadata: name: nginx labels: app: nginx spec: containers: - name: nginx image: nginx ports: - containerPort: 80 ``` 

kubectl apply -f nginx-pod.yaml 此时k8s会在所有可用的Node上，创建3个Pod，并且每个Pod都有一个app: nginx的label，同时每个Pod中都运行了一个nginx容器。 如果某个Pod发生问题，Controller Manager能够及时发现，然后根据RC的定义，创建一个新的Pod 扩缩容：kubectl scale rc nginx --replicas=5


ReplicaSets
在Kubernetes v1.2时，RC就升级成了另外一个概念：Replica Set，官方解释为“下一代RC”

ReplicaSet和RC没有本质的区别，kubectl中绝大部分作用于RC的命令同样适用于RS

RS与RC唯一的区别是：RS支持基于集合的Label Selector（Set-based selector），
而RC只支持基于等式的Label Selector（equality-based selector），这使得Replica Set的功能更强
官网解释
A ReplicaSet’s purpose is to maintain a stable set of replica Pods running at any given time. 
As such, it is often used to guarantee the availability of a specified number of identical Pods.
案例

```yaml
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
  name: frontend
spec:
  matchLabels: #这个就是Label selector
    tier: frontend
  matchExpressions: 
    - {key:tier,operator: In,values: [frontend]}
  template:
  ...
  ...
```
注意：一般情况下，我们很少单独使用Replica Set，它主要是被Deployment这个更高的资源对象所使用，从而形成一整套Pod创建、删除、更新的编排机制。当我们使用Deployment时，无须关心它是如何创建和维护Replica Set的，这一切都是自动发生的。同时，无需担心跟其他机制的不兼容问题（比如ReplicaSet不支持rolling-update但Deployment支持）。


Deployment
Deployment相对RC最大的一个升级就是我们可以随时知道当前Pod“部署”的进度。

创建一个Deployment对象来生成对应的Replica Set并完成Pod副本的创建过程

检查Deploymnet的状态来看部署动作是否完成（Pod副本的数量是否达到预期的值）
官网解释：
A Deployment provides declarative (陈述的) updates for Pods and ReplicaSets.

You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.
案例

\```yaml apiVersion: apps/v1 kind: Deployment metadata: name: nginx-deployment # selector能够选择对应的pod的label labels: app: nginx spec: replicas: 3 selector: matchLabels: # 这里是一个RS，匹配具有同一个label属性的pod标签 app: nginx template: # 定义pod的模板 metadata: labels: app: nginx # 定义当前pod的label属性，app为key，value为nginx。 spec: containers: - name: nginx image: nginx:1.7.9 ports: - containerPort: 80 ```


StatefulSet
`官网`：<https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/>
StatefulSet is the workload API object used to manage stateful applications.
Manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods.
之前接触的Pod的管理对象比如RC、Deployment、DaemonSet和Job都是面向无状态的服务，但是现实中有很多服务是有状态的，比如MySQL集群、MongoDB集群、ZK集群等，它们都有以下共同的特点：
\> - 每个节点都有固定的ID，通过该ID，集群中的成员可以互相发现并且通信
\> - 集群的规模是比较固定的，集群规模不能随意变动
\> - 集群里的每个节点都是有状态的，通常会持久化数据到永久存储中
\> - 如果磁盘损坏，则集群里的某个节点无法正常运行，集群功能受损

而之前的RC/Deployment没办法满足要求，所以从Kubernetes v1.4版本就引入了PetSet资源对象，在v1.5版本时更名为StatefulSet。从本质上说，StatefulSet可以看作是Deployment/RC对象的特殊变种
\- StatefulSet里的每个Pod都有稳定、唯一的网络标识，可以用来发现集群内其他的成员
\> - Pod的启动顺序是受控的，操作第n个Pod时，前n-1个Pod已经是运行且准备好的状态
\> - StatefulSet里的Pod采用稳定的持久化存储卷，通过PV/PVC来实现，删除Pod时默认不会删除与StatefulSet相关的存储卷
\> - StatefulSet需要与Headless Service配合使用
实例

\> kubectl apply nginx-st.yaml > > watch kubectl get pods # 观察pod的创建顺序，以及pod的名字 ```yaml # 定义Service apiVersion: v1 kind: Service metadata: name: nginx labels: app: nginx spec: ports: - port: 80 name: web clusterIP: None selector: app: nginx --- # 定义StatefulSet apiVersion: apps/v1 kind: StatefulSet metadata: name: web spec: selector: matchLabels: app: nginx serviceName: "nginx" replicas: 3 template: metadata: labels: app: nginx spec: terminationGracePeriodSeconds: 10 containers: - name: nginx image: nginx ports: - containerPort: 80 name: web ```


DaemonSet
`官网`：https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/
A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.
DaemonSet应用场景
\> - 运行集群存储 daemon，例如在每个节点上运行 `glusterd`、`ceph`。
\> - 在每个节点上运行日志收集 daemon，例如`fluentd`、`logstash`。
\> - 在每个节点上运行监控 daemon，例如 [Prometheus Node Exporter](https://github.com/prometheus/node_exporter)、`collectd`、Datadog 代理、New Relic 代理，或 Ganglia `gmond`。


任务集


Job
官网：https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/
A Job creates one or more Pods and ensures that a specified number of them successfully terminate. 
As pods successfully complete, the Job tracks the successful completions. 
When a specified number of successful completions is reached, the task (ie, Job) is complete. 
Deleting a Job will clean up the Pods it created.
对于RS，RC之类的控制器，能够保持Pod按照预期数目持久地运行下去，它们针对的是持久性的任务，比如web服务。
而有些操作其实不需要持久，比如压缩文件，我们希望任务完成之后，Pod就结束运行，不需要保持在系统中，此时就需要用到Job。
所以可以这样理解，Job是对RS、RC等持久性控制器的补充。

负责批量处理短暂的一次性任务，仅执行一次，并保证处理的一个或者多个Pod成功结束。
实例

\```yaml apiVersion: batch/v1 kind: Job metadata: name: job-demo spec: template: metadata: name: job-demo spec: restartPolicy: Never containers: - name: counter image: busybox command: - "bin/sh" - "-c" - "for i in 9 8 7 6 5 4 3 2 1; do echo $i; done" ``` > kubectl apply -f job.yaml > > kubectl describe jobs/pi > > kubectl logs pod-name
\- 非并行Job:
\- 通常只运行一个Pod，Pod成功结束Job就退出。
\- 固定完成次数的并行Job:
\- 并发运行指定数量的Pod，直到指定数量的Pod成功，Job结束。
\- 带有工作队列的并行Job:
\- 用户可以指定并行的Pod数量，当任何Pod成功结束后，不会再创建新的Pod
\- 一旦有一个Pod成功结束，并且所有的Pods都结束了，该Job就成功结束。
\- 一旦有一个Pod成功结束，其他Pods都会准备退出。


CronJob
`官网`：https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

A Cron Job creates Jobs on a time-based schedule.
One CronJob object is like one line of a crontab (cron table) file. It runs a job periodically on a given schedule, written in Cron format.
cronJob是基于时间进行任务的定时管理。
在特定的时间点运行任务
\- 反复在指定的时间点运行任务：比如定时进行数据库备份，定时发送电子邮件等等。


Horizontal Pod Autoscaler
`官网`：https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
The Horizontal Pod Autoscaler automatically scales the number of pods in a replication controller, deployment or replica set based on observed CPU utilization (or, with custom metrics support, on some other application-provided metrics). Note that Horizontal Pod Autoscaling does not apply to objects that can’t be scaled, for example, DaemonSets.
使用Horizontal Pod Autoscaling，Kubernetes会自动地根据观察到的CPU利用率(或者通过一些其他应用程序提供的自定义的指标)自动地缩放在replication controller、deployment或replica set上pod的数量。
实例

\> (0)前期准备 > > kubectl apply -f nginx-deployment.yaml > > ```yaml > apiVersion: apps/v1 > kind: Deployment > metadata: > name: nginx-deployment > labels: > app: nginx > spec: > replicas: 3 > selector: > matchLabels: > app: nginx > template: > metadata: > labels: > app: nginx > spec: > containers: > - name: nginx > image: nginx > ports: > - containerPort: 80 > ``` > (1)创建hpa ```shell # 使nginx pod的数量介于2和10之间，CPU使用率维持在50％ kubectl autoscale deployment nginx-deployment --min=2 --max=10 --cpu-percent=50 ``` > (2)查看所有创建的资源 ```shell kubectl get pods kubectl get deploy kubectl get hpa ``` > (3)修改replicas值为1或者11 > > 可以发现最终最小还是2，最大还是10 ```shell kubectl edit deployment nginx-deployment ``` > (4)再次理解什么是hpa > > ``` > Horizontal Pod Autoscaling可以根据CPU使用率或应用自定义metrics自动扩展Pod数量（支持replication controller、deployment和replica set） > ``` > > ``` > 01-控制管理器每隔30s查询metrics的资源使用情况 > 02-通过kubectl创建一个horizontalPodAutoscaler对象，并存储到etcd中 > 03-APIServer:负责接受创建hpa对象，然后存入etcd > ```


Resource
requests
limits
实例

\### 3.1 Resource > 因为K8S的最小操作单元是Pod，所以这里主要讨论的是Pod的资源 > > `官网`： > > 在K8S的集群中，Node节点的资源信息会上报给APIServer > requests&limits > > 可以通过这两个属性设置cpu和内存 > > ``` > When Containers have resource requests specified, the scheduler can make better decisions about which nodes to place Pods on. And when Containers have their limits specified, contention for resources on a node can be handled in a specified manner. > ``` > > ```yaml > apiVersion: v1 > kind: Pod > metadata: > name: frontend > spec: > containers: > - name: db > image: mysql > env: > - name: MYSQL_ROOT_PASSWORD > value: "password" > resources: > requests: > memory: "64Mi" # 表示64M需要内存 > cpu: "250m" # 表示需要0.25核的CPU > limits: > memory: "128Mi" > cpu: "500m" > - name: wp > image: wordpress > resources: > requests: > memory: "64Mi" > cpu: "250m" > limits: > memory: "128Mi" > cpu: "500m" > ```


Dashboard
Dashboard is a web-based Kubernetes user interface. You can use Dashboard to deploy containerized applications to a Kubernetes cluster, troubleshoot your containerized application, and manage the cluster resources. You can use Dashboard to get an overview of applications running on your cluster, as well as for creating or modifying individual Kubernetes resources (such as Deployments, Jobs, DaemonSets, etc). For example, you can scale a Deployment, initiate a rolling update, restart a pod or deploy new applications using a deploy wizard.
一个界面控制端


pod生命周期
init container


main contianer
post start hook :容器启动后做什么操作
readiness probe ：容器正常工作阶段
pre stop hook ：容器停止前做什么操作


探针 Container probes


livenessProbe


Handlers
ExecAction
TCPSocketAction
HTTPGetAction


readinessProbe


Handlers
ExecAction
TCPSocketAction
HTTPGetAction


重启策略 restartPolicy
1、Always：但凡pod对象终止就将其重启，此为默认设定
2、OnFailure：尽在pod对象出现错误时方才将其重启
3、Never：从不重启。


pod对象的状态
1.pending：Pod 已被 Kubernetes 系统接受，但有一个或者多个容器镜像尚未创建。等待时间包括调度 
Pod 的时间和通过网络下载镜像的时间，这可能需要花点时间。
2.running： pod 已经被调度到某节点，并且所有容器都已经被kubectl创建完成。
3.succeeded：pod中的所有容器都已经成功终止并且不会被重启；
4.failed：所有容器都已经终止，但至少有一个容器终止失败，即容器返回了非0值的推出状态或已经被系统终止。
5.unknown：apiserver无法正常获取到pod对象的状态信息，通常是由于其无法与所在工作节点的kubelet通信所致。


通信模式


网络通信模式
官网原话：
Each Pod is assigned a unique IP address. 
Every container in a Pod shares the network namespace, 
including the IP address and network ports.
同一个pod中的容器是共享网络ip地址和端口号的，可以直接进行通信
那如果是通过容器的名称进行通信呢？就需要将所有pod中的容器加入到同一个容器的网络中，
我们把该容器称作为pod中的pause container。


集群内Pod之间的通信
Calico——K8s的网络model

\- pods on a node can communicate with all pods on all nodes without NAT
\- agents on a node (e.g. system daemons, kubelet) can communicate with all pods on that node
\- pods in the host network of a node can communicate with all pods on all nodes without NAT

准备两个pod，一个nginx，一个busybox > nginx_pod.yaml > > ```yaml > apiVersion: v1 > kind: Pod > metadata: > name: nginx-pod > labels: > app: nginx > spec: > containers: > - name: nginx-container > image: nginx > ports: > - containerPort: 80 > ``` > busybox_pod.yaml > > ```yaml > apiVersion: v1 > kind: Pod > metadata: > name: busybox > labels: > app: busybox > spec: > containers: > - name: busybox > image: busybox > command: ['sh', '-c', 'echo The app is running! && sleep 3600'] > ``` > 将两个pod运行起来，并且查看运行情况 > > kubectl apply -f nginx_pod.yaml > > kubectl apply -f busy_pod.yaml > > kubectl get pods -o wide > > ``` > NAME READY STATUS RESTARTS AGE IP NODE > busybox 1/1 Running 0 49s 192.168.221.70 worker02-kubeadm-k8s > nginx-pod 1/1 Running 0 7m46s 192.168.14.1 worker01-kubeadm-k8s > ``` > > `发现`：nginx-pod的ip为192.168.14.1 busybox-pod的ip为192.168.221.70 #### 同一个集群中同一台机器 > (1)来到worker01：ping 192.168.14.1 ```shell PING 192.168.14.1 (192.168.14.1) 56(84) bytes of data. 64 bytes from 192.168.14.1: icmp_seq=1 ttl=64 time=0.063 ms 64 bytes from 192.168.14.1: icmp_seq=2 ttl=64 time=0.048 ms ``` > (2)来到worker01：curl 192.168.14.1 ```html ``` #### 同一个集群中不同机器 > (1)来到worker02：ping 192.168.14.1 ```shell [root@worker02-kubeadm-k8s ~]# ping 192.168.14.1 PING 192.168.14.1 (192.168.14.1) 56(84) bytes of data. 64 bytes from 192.168.14.1: icmp_seq=1 ttl=63 time=0.680 ms 64 bytes from 192.168.14.1: icmp_seq=2 ttl=63 time=0.306 ms 64 bytes from 192.168.14.1: icmp_seq=3 ttl=63 time=0.688 ms ``` > (2)来到worker02：curl 192.168.14.1，同样可以访问nginx > (3)来到master： > > ping/curl 192.168.14.1 访问的是worker01上的nginx-pod > > ping 192.168.221.70 访问的是worker02上的busybox-pod > (4)来到worker01：ping 192.168.221.70 访问的是worker02上的busybox-pod
结论：通过Calico等网络，K8s的Pods可以跨机器访问，但是IP会是变动的。


集群内的Service-Cluster IP
把相同或者具有关联的Pod，打上Label，组成Service。
而Service有固定的IP，不管Pod怎么创建和销毁，都可以通过Service的IP进行访问
Service官网解释：https://kubernetes.io/docs/concepts/services-networking/service/

An abstract way to expose an application running on a set of Pods as a network service.

With Kubernetes you don’t need to modify your application to use an unfamiliar service
 discovery mechanism. Kubernetes gives Pods their own IP addresses and a single DNS 
name for a set of Pods, and can load-balance across them.
案例

\> 对于上述的Pod虽然实现了集群内部互相通信，但是Pod是不稳定的，比如通过Deployment管理Pod，随时可能对Pod进行扩缩容，这时候Pod的IP地址是变化的。能够有一个固定的IP，使得集群内能够访问。也就是之前在架构描述的时候所提到的，能够把相同或者具有关联的Pod，打上Label，组成Service。而Service有固定的IP，不管Pod怎么创建和销毁，都可以通过Service的IP进行访问 > > `Service官网`： > > ``` > An abstract way to expose an application running on a set of Pods as a network service. > With Kubernetes you don’t need to modify your application to use an unfamiliar service discovery mechanism. Kubernetes gives Pods their own IP addresses and a single DNS name for a set of Pods, and can load-balance across them. > ``` > (1)创建whoami-deployment.yaml文件，并且apply ```yaml apiVersion: apps/v1 kind: Deployment metadata: name: whoami-deployment labels: app: whoami spec: replicas: 3 selector: matchLabels: app: whoami template: metadata: labels: app: whoami spec: containers: - name: whoami image: jwilder/whoami ports: - containerPort: 8000 ``` > (2)查看pod以及service ``` whoami-deployment-5dd9ff5fd8-22k9n 192.168.221.80 worker02-kubeadm-k8s whoami-deployment-5dd9ff5fd8-vbwzp 192.168.14.6 worker01-kubeadm-k8s whoami-deployment-5dd9ff5fd8-zzf4d 192.168.14.7 worker01-kubeadm-k8s ``` kubect get svc:可以发现目前并没有关于whoami的service ``` NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE kubernetes ClusterIP 10.96.0.1443/TCP 19h ``` > (3)在集群内正常访问 ``` curl 192.168.221.80:8000/192.168.14.6:8000/192.168.14.7:8000 ``` > (4)创建whoami的service > > `注意`：该地址只能在集群内部访问 ``` kubectl expose deployment whoami-deployment kubectl get svc 删除svc kubectl delete service whoami-deployment [root@master-kubeadm-k8s ~]# kubectl get svc NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE kubernetes ClusterIP 10.96.0.1443/TCP 19h whoami-deployment ClusterIP 10.105.147.59 8000/TCP 23s ``` 可以发现有一个Cluster IP类型的service，名称为whoami-deployment，IP地址为10.101.201.192 > (5)通过Service的Cluster IP访问 ``` [root@master-kubeadm-k8s ~]# curl 10.105.147.59:8000 I'm whoami-deployment-678b64444d-b2695 [root@master-kubeadm-k8s ~]# curl 10.105.147.59:8000 I'm whoami-deployment-678b64444d-hgdrk [root@master-kubeadm-k8s ~]# curl 10.105.147.59:8000 I'm whoami-deployment-678b64444d-65t88 ``` > (6)具体查看一下whoami-deployment的详情信息，发现有一个Endpoints连接了具体3个Pod ``` [root@master-kubeadm-k8s ~]# kubectl describe svc whoami-deployment Name: whoami-deployment Namespace: default Labels: app=whoami Annotations: Selector: app=whoami Type: ClusterIP IP: 10.105.147.59 Port: 8000/TCP TargetPort: 8000/TCP Endpoints: 192.168.14.8:8000,192.168.221.81:8000,192.168.221.82:8000 Session Affinity: None Events: ``` > (7)不妨对whoami扩容成5个 ``` kubectl scale deployment whoami-deployment --replicas=5 ``` > (8)再次访问：curl 10.105.147.59:8000 > (9)再次查看service具体信息：kubectl describe svc whoami-deployment > (10)其实对于Service的创建，不仅仅可以使用kubectl expose，也可以定义一个yaml文件 ```yaml apiVersion: v1 kind: Service metadata: name: my-service spec: selector: app: MyApp ports: - protocol: TCP     port: 80 targetPort: 9376 type: Cluster ``` `conclusion`：其实Service存在的意义就是为了Pod的不稳定性，而上述探讨的就是关于Service的一种类型Cluster IP，只能供集群内访问 > 以Pod为中心，已经讨论了关于集群内的通信方式，接下来就是探讨集群中的Pod访问外部服务，以及外部服务访问集群中的Pod


实现方式
\1. kubectl expose
kubectl describe svc whoami-deployment
yaml文件定义Service类型

\```yaml apiVersion: v1 kind: Service metadata: name: my-service # service名称 spec: selector: #选择哪些pod app: MyApp ports: - protocol: TCP port: 80 targetPort: 9376 type: Cluster ```
Service存在的意义就是为了Pod的不稳定性，
而上述探讨的就是关于Service的一种类型Cluster IP，只能供集群内访问
外部访问，直接通过expose的服务的ip地址访问即可。


外部服务访问集群之中的Pod
Service-NodePort

\#### Service-NodePort > 也是Service的一种类型，可以通过NodePort的方式 > > 说白了，因为外部能够访问到集群的物理机器IP，所以就是在集群中每台物理机器上暴露一个相同的IP物理端口，比如32008 > (1)根据whoami-deployment.yaml创建pod ```yaml apiVersion: apps/v1 kind: Deployment metadata: name: whoami-deployment labels: app: whoami spec: replicas: 3 selector: matchLabels: app: whoami template: metadata: labels: app: whoami spec: containers: - name: whoami image: jwilder/whoami ports: - containerPort: 8000 ``` > (2)创建NodePort类型的service，名称为whoami-deployment ``` kubectl delete svc whoami-deployment kubectl expose deployment whoami-deployment --type=NodePort [root@master-kubeadm-k8s ~]# kubectl get svc NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE kubernetes ClusterIP 10.96.0.1 443/TCP 21h whoami-deployment NodePort 10.99.108.82 8000:32041/TCP 7s ``` > (3)注意上述的端口32041，实际上就是暴露在集群中物理机器上的端口 ``` lsof -i tcp:32041 netstat -ntlp|grep 32041 ``` > (4)浏览器通过物理机器的IP访问 ``` http://192.168.0.51:32041 curl 192.168.0.61:32041 ``` `conclusion`：NodePort虽然能够实现外部访问Pod的需求，但是真的好吗？其实不好，占用了各个物理主机上的端口


Service-LoadBalance
通常需要第三方云提供商支持，有约束性


Ingress

vi my-tomcat.yaml kubectl apply -f my-tomcat.yaml kubectl get pods kubectl get deployment kubectl get svc `tomcat-service NodePort 10.105.51.9780:31032/TCP 37s` ```yaml apiVersion: apps/v1 kind: Deployment metadata:  name: tomcat-deployment labels: app: tomcat spec: replicas: 1 selector: matchLabels: app: tomcat template: metadata: labels: app: tomcat spec: containers: - name: tomcat image: tomcat ports: - containerPort: 8080 --- apiVersion: v1 kind: Service metadata: name: tomcat-service spec: ports: - port: 80 protocol: TCP targetPort: 8080 selector: app: tomcat type: NodePort ``` > 显然，Service-NodePort的方式生产环境不推荐使用，那接下来就基于上述需求，使用Ingress实现访问tomcat的需求。 > > `官网Ingress`: > > `GitHub Ingress Nginx`: > > `Nginx Ingress Controller`: (1)以Deployment方式创建Pod，该Pod为Ingress Nginx Controller，要想让外界访问，可以通过Service的NodePort或者HostPort方式，这里选择HostPort，比如指定worker01运行 ```shell # 确保nginx-controller运行到w1节点上 kubectl label node w1 name=ingress # 使用HostPort方式运行，需要增加配置 hostNetwork: true # 搜索nodeSelector，并且要确保w1节点上的80和443端口没有被占用，镜像拉取需要较长的时间，这块注意一下哦 # mandatory.yaml在网盘中的“课堂源码”目录 kubectl apply -f mandatory.yaml kubectl get all -n ingress-nginx ``` > (2)查看w1的80和443端口 ``` lsof -i tcp:80 lsof -i tcp:443 ``` > (3)创建tomcat的pod和service > > > 记得将之前的tomcat删除：kubectl delete -f my-tomcat.yaml > > vi tomcat.yaml > > kubectl apply -f tomcat.yaml > > kubectl get svc > > kubectl get pods ```yaml apiVersion: apps/v1 kind: Deployment metadata: name: tomcat-deployment labels: app: tomcat spec: replicas: 1 selector: matchLabels: app: tomcat template: metadata: labels: app: tomcat spec: containers: - name: tomcat image: tomcat ports: - containerPort: 8080 --- apiVersion: v1 kind: Service metadata: name: tomcat-service spec: ports: - port: 80 protocol: TCP targetPort: 8080 selector: app: tomcat ``` > (4)创建Ingress以及定义转发规则 > > kubectl apply -f nginx-ingress.yaml > > kubectl get ingress > > kubectl describe ingress nginx-ingress ```yaml #ingress apiVersion: extensions/v1beta1 kind: Ingress metadata: name: nginx-ingress spec: rules: - host: tomcat.jack.com http: paths: - path: / backend: serviceName: tomcat-service servicePort: 80 ``` > (5)修改win的hosts文件，添加dns解析 ``` 192.168.8.61 tomcat.jack.com ``` > (6)打开浏览器，访问tomcat.jack.com `总结`：如果以后想要使用Ingress网络，其实只要定义ingress，service和pod即可，前提是要保证nginx ingress controller已经配置好了。
官网解释： 
An API object that manages external access to the services in a cluster, typically HTTP.

Ingress can provide load balancing, SSL termination and name-based virtual hosting.
Service HostPort

显然，Service-NodePort的方式生产环境不推荐使用，那接下来就基于上述需求，使用Ingress实现访问tomcat的需求。 > `官网Ingress`: > > `GitHub Ingress Nginx`: > > `Nginx Ingress Controller`: (1)以Deployment方式创建Pod，该Pod为Ingress Nginx Controller，要想让外界访问，可以通过Service的NodePort或者HostPort方式，这里选择HostPort，比如指定worker01运行 ```shell # 确保nginx-controller运行到w1节点上 kubectl label node w1 name=ingress # 使用HostPort方式运行，需要增加配置 hostNetwork: true # 搜索nodeSelector，并且要确保w1节点上的80和443端口没有被占用，镜像拉取需要较长的时间，这块注意一下哦 # mandatory.yaml在网盘中的“课堂源码”目录 kubectl apply -f mandatory.yaml kubectl get all -n ingress-nginx ``` mandatory.yaml ```YAML apiVersion: v1 kind: Namespace metadata: name: ingress-nginx labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx --- kind: ConfigMap apiVersion: v1 metadata: name: nginx-configuration namespace: ingress-nginx labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx --- kind: ConfigMap apiVersion: v1 metadata: name: tcp-services namespace: ingress-nginx labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx --- kind: ConfigMap apiVersion: v1 metadata: name: udp-services namespace: ingress-nginx labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx --- apiVersion: v1 kind: ServiceAccount metadata: name: nginx-ingress-serviceaccount namespace: ingress-nginx labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx --- apiVersion: rbac.authorization.k8s.io/v1beta1 kind: ClusterRole metadata: name: nginx-ingress-clusterrole labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx rules: - apiGroups: - "" resources: - configmaps - endpoints - nodes - pods - secrets verbs: - list - watch - apiGroups: - "" resources: - nodes verbs: - get - apiGroups: - "" resources: - services verbs: - get - list - watch - apiGroups: - "" resources: - events verbs: - create - patch - apiGroups: - "extensions" - "networking.k8s.io" resources: - ingresses verbs: - get - list - watch - apiGroups: - "extensions" - "networking.k8s.io" resources: - ingresses/status verbs: - update --- apiVersion: rbac.authorization.k8s.io/v1beta1 kind: Role metadata: name: nginx-ingress-role namespace: ingress-nginx labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx rules: - apiGroups: - "" resources: - configmaps - pods - secrets - namespaces verbs: - get - apiGroups: - "" resources: - configmaps resourceNames: # Defaults to "-" # Here: "-" # This has to be adapted if you change either parameter # when launching the nginx-ingress-controller. - "ingress-controller-leader-nginx" verbs: - get - update - apiGroups: - "" resources: - configmaps verbs: - create - apiGroups: - "" resources: - endpoints verbs: - get --- apiVersion: rbac.authorization.k8s.io/v1beta1 kind: RoleBinding metadata: name: nginx-ingress-role-nisa-binding namespace: ingress-nginx labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx roleRef: apiGroup: rbac.authorization.k8s.io kind: Role name: nginx-ingress-role subjects: - kind: ServiceAccount name: nginx-ingress-serviceaccount namespace: ingress-nginx --- apiVersion: rbac.authorization.k8s.io/v1beta1 kind: ClusterRoleBinding metadata: name: nginx-ingress-clusterrole-nisa-binding labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx roleRef: apiGroup: rbac.authorization.k8s.io kind: ClusterRole name: nginx-ingress-clusterrole subjects: - kind: ServiceAccount name: nginx-ingress-serviceaccount namespace: ingress-nginx --- apiVersion: apps/v1 kind: Deployment metadata:  name: nginx-ingress-controller namespace: ingress-nginx labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx spec: replicas: 1 selector: matchLabels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx template: metadata: labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx annotations: prometheus.io/port: "10254" prometheus.io/scrape: "true" spec: # wait up to five minutes for the drain of connections terminationGracePeriodSeconds: 300 serviceAccountName: nginx-ingress-serviceaccount hostNetwork: true #设置为HostPort nodeSelector: name: ingress kubernetes.io/os: linux containers: - name: nginx-ingress-controller image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.26.1 args: - /nginx-ingress-controller - --configmap=$(POD_NAMESPACE)/nginx-configuration - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services - --udp-services-configmap=$(POD_NAMESPACE)/udp-services - --publish-service=$(POD_NAMESPACE)/ingress-nginx - --annotations-prefix=nginx.ingress.kubernetes.io securityContext: allowPrivilegeEscalation: true capabilities: drop: - ALL add: - NET_BIND_SERVICE # www-data -> 33 runAsUser: 33 env: - name: POD_NAME valueFrom: fieldRef: fieldPath: metadata.name - name: POD_NAMESPACE valueFrom: fieldRef: fieldPath: metadata.namespace ports: - name: http containerPort: 80 - name: https containerPort: 443 livenessProbe: failureThreshold: 3 httpGet: path: /healthz port: 10254 scheme: HTTP initialDelaySeconds: 10 periodSeconds: 10 successThreshold: 1 timeoutSeconds: 10 readinessProbe: failureThreshold: 3 httpGet: path: /healthz port: 10254 scheme: HTTP periodSeconds: 10 successThreshold: 1 timeoutSeconds: 10 lifecycle: preStop: exec: command: - /wait-shutdown --- ``` > (2)查看w1的80和443端口 ``` lsof -i tcp:80 lsof -i tcp:443 ``` > (3)创建tomcat的pod和service > > > 记得将之前的tomcat删除：kubectl delete -f my-tomcat.yaml > > vi tomcat.yaml > > kubectl apply -f tomcat.yaml > > kubectl get svc > > kubectl get pods ```yaml apiVersion: apps/v1 kind: Deployment metadata: name: tomcat-deployment labels: app: tomcat spec: replicas: 1 selector: matchLabels: app: tomcat template: metadata: labels: app: tomcat spec: containers: - name: tomcat image: tomcat ports: - containerPort: 8080 --- apiVersion: v1 kind: Service metadata: name: tomcat-service spec: ports: - port: 80 protocol: TCP targetPort: 8080 selector: app: tomcat ``` > (4)创建Ingress以及定义转发规则 > > kubectl apply -f nginx-ingress.yaml > > kubectl get ingress > > kubectl describe ingress nginx-ingress ```yaml #ingress apiVersion: extensions/v1beta1 kind: Ingress metadata: name: nginx-ingress spec: rules: - host: tomcat.jack.com http: paths: - path: / backend: serviceName: tomcat-service servicePort: 80 ``` > (5)修改win的hosts文件，添加dns解析 ``` 192.168.8.61 tomcat.jack.com ``` > (6)打开浏览器，访问tomcat.jack.com `总结`：如果以后想要使用Ingress网络，其实只要定义ingress，service和pod即可，前提是要保证nginx ingress controller已经配置好了。
通过yaml文件指定

\```yaml #ingress apiVersion: extensions/v1beta1 kind: Ingress metadata: name: nginx-ingress spec: rules: - host: tomcat.xiaohei.com http: paths: - path: / backend: serviceName: tomcat-service servicePort: 80 ```


Nginx
Http 代理访问
Https代理访问
使用cookie 实现会话关联
BasicAuth
Nginx进行重写
组件通信模式


namespaces


命名空间
命名空间就是为了隔离不同的资源，比如：Pod、Service、Deployment等。
可以在输入命令的时候指定命名空间-n，如果不指定，则使用默认的命名空间：default。
案例：创建命名空间

myns-namespace.yaml ```yaml apiVersion: v1 kind: Namespace metadata: name: myns ``` kubectl apply -f myns-namespace.yaml kubectl get namespaces ``` NAME STATUS AGE default Active 50m kube-node-lease Active 50m kube-public Active 50m kube-system Active 50m myns Active 27s ```
案例：指定命名空间下的资源

比如创建一个pod，属于myns命名空间下 vi nginx-pod.yaml kubectl apply -f nginx-pod.yaml ```yaml apiVersion: v1 kind: Pod metadata: name: nginx-pod namespace: myns #指定namespace spec: containers: - name: nginx-container   image: nginx ports: - containerPort: 80 ``` 查看myns命名空间下的Pod和资源 kubectl get pods kubectl get pods -n myns kubectl get all -n myns kubectl get pods --all-namespaces #查找所有命名空间下的pod


## 资源清单


资源的概念
什么是资源
名称空间级别的资源
集群级别的资源


资源清单


yaml 文件
yaml 语法格式
通过资源清单编写Pod
Pod的生命周期


## 储存


volume


定义概念
卷的类型
官网解释
On-disk files in a Container are ephemeral, which presents some problems for 
nontrivial applications when running in Containers. 
First, when a Container crashes, kubelet will restart it, but the files will be lost - 
the Container starts with a clean state. 
Second, when running Containers together in a Pod it is often necessary to 
share files between those Containers. 
The Kubernetes Volume abstraction solves both of these problems. 


分类


Host Volume
实例

\### Host类型volume实战 > `背景`：定义一个Pod，其中包含两个Container，都使用Pod的Volume > > volume-pod.yaml ```yaml apiVersion: v1 kind: Pod metadata: name: volume-pod spec: containers: - name: nginx-container image: nginx ports: - containerPort: 80 volumeMounts: # volume数据 - name: volume-pod mountPath: /nginx-volume - name: busybox-container image: busybox command: ['sh', '-c', 'echo The app is running! && sleep 3600'] volumeMounts: - name: volume-pod mountPath: /busybox-volume volumes: - name: volume-pod hostPath: path: /tmp/volume-pod ``` > (1)创建资源 ``` kubectl apply -f volume-pod.yaml ``` > (2)查看pod的运行情况 ``` kubectl get pods -o wide ``` > (3)来到运行的worker节点 ```shell docker ps | grep volume ls /tmp/volume-pod docker exec -it containerid sh ls /nginx-volume ls /busybox-volume # 折腾一下是否同步 ``` > (4)查看pod中的容器里面的hosts文件，是否一样。 > > 发现是一样的，并且都是由pod管理的 ``` docker exec -it containerid cat /etc/hosts ``` > (5)所以一般container中的存储或者网络的内容，不要在container层面修改，而是在pod中修改 > > 比如下面修改一下网络 > > ```yaml > spec: > hostNetwork: true > hostPID: true > hostAliases: > - ip: "192.168.8.61" > hostnames: > - "test.jack.com" > containers: > - name: nginx-contain	er > image: nginx > ```


PV


Persistent Volume
实例

\### PersistentVolume > `官网`：https://kubernetes.io/docs/concepts/storage/persistent-volumes/ ```yaml apiVersion: v1 kind: PersistentVolume metadata: name: my-pv spec: capacity: storage: 5Gi # 存储空间大小 volumeMode: Filesystem accessModes: - ReadWriteOnce # 只允许一个Pod进行独占式读写操作 persistentVolumeReclaimPolicy: Recycle storageClassName: slow mountOptions: - hard - nfsvers=4.1 nfs: path: /tmp # 远端服务器的目录 server: 172.17.0.2 # 远端的服务器 ``` > 说白了，PV是K8s中的资源，volume的plugin实现，生命周期独立于Pod，封装了底层存储卷实现的细节。 > > `注意`：PV的维护通常是由运维人员、集群管理员进行维护的。
PV是K8s中的资源，volume的plugin实现，生命周期独立于Pod，封装了底层存储卷实现的细节。

注意：PV的维护通常是由运维人员、集群管理员进行维护的。
后端类型
PV访问策略说明


回收策略说明
Retain：表示删除PVC的时候，PV不会一起删除，而是变成Released状态等待管理员手动清理

Recycle：在Kubernetes新版本就不用了，采用动态PV供给来替代

Delete：表示删除PVC的时候，PV也会一起删除，同时也删除PV所指向的实际存储空间

注意：目前只有NFS和HostPath支持Recycle策略。AWS EBS、GCE PD、Azure Disk和Cinder支持Delete策略


状态
Available：表示当前的pv没有被绑定

Bound：表示已经被pvc挂载

Released：pvc没有在使用pv, 需要管理员手工释放pv

Failed：资源回收失败
实例演示


PVC
PersistentVolumeClaim
官网：https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims
有了PV，那Pod如何使用呢？为了方便使用，我们可以设计出一个PVC来绑定PV，然后把PVC交给Pod来使用即可。
用例

\> ```yaml apiVersion: v1 kind: PersistentVolumeClaim metadata: name: myclaim spec: accessModes: - ReadWriteOnce volumeMode: Filesystem resources: requests: storage: 8Gi storageClassName: slow selector: matchLabels: release: "stable" matchExpressions: - {key: environment, operator: In, values: [dev]} ``` > 说白了，PVC会匹配满足要求的PV[是根据size和访问模式进行匹配的]，进行一一绑定，然后它们的状态都会变成Bound。 > > 也就是PVC负责请求PV的大小和访问方式，然后Pod中就可以直接使用PVC咯。 > > `注意`：PVC通常由开发小伙伴维护，开发小伙伴无需关注与存储细节。 Pod之中使用PVC ```YAML apiVersion: v1 kind: Pod metadata: name: mypod spec: containers: - name: myfrontend image: nginx volumeMounts: - mountPath: "/var/www/html" name: mypd volumes: - name: mypd persistentVolumeClaim: claimName: myclaim ```
ngnix使用pvc实战

\### Pod中使用PVC实战 > `背景`：使用nginx持久化存储演示 > > ``` > (1)共享存储使用nfs，比如选择在m节点 > (2)创建pv和pvc > (3)nginx pod中使用pvc > ``` #### 1.6.1 master节点搭建nfs > 在master节点上搭建一个NFS服务器，目录为/nfs/data > > ```shell > nfs(network file system)网络文件系统，是FreeBSD支持的文件系统中的一种，允许网络中的计算机之间通过TCP/IP网络共享资源 > > 01 选择master节点作为nfs的server，所以在master节点上 > # 安装nfs > yum install -y nfs-utils > # 创建nfs目录 > mkdir -p /nfs/data/ > mkdir -p /nfs/data/mysql > # 授予权限 > chmod -R 777 /nfs/data > # 编辑export文件 > vi /etc/exports > /nfs/data (rw,no_root_squash,sync) > # 使得配置生效 > exportfs -r > # 查看生效 > exportfs > # 启动rpcbind、nfs服务 > systemctl restart rpcbind && systemctl enable rpcbind > systemctl restart nfs && systemctl enable nfs > # 查看rpc服务的注册情况 > rpcinfo -p localhost > # showmount测试 > showmount -e master-ip > > 02 所有node上安装客户端 > yum -y install nfs-utils > systemctl start nfs && systemctl enable nfs > ``` #### 1.6.2 创建PV&PVC&Nginx > (1)在nfs服务器创建所需要的目录 > > mkdir -p /nfs/data/nginx > (2)定义PV，PVC和Nginx的yaml文件 > ```YAML # 定义PV apiVersion: v1 kind: PersistentVolume metadata: name: nginx-pv spec:  accessModes: - ReadWriteMany capacity: storage: 2Gi nfs: path: /nfs/data/nginx server: 121.41.10.13 --- # 定义PVC，用于消费PV apiVersion: v1 kind: PersistentVolumeClaim metadata: name: nginx-pvc spec: accessModes: - ReadWriteMany resources: requests: storage: 2Gi --- # 定义Pod，指定需要使用的PVC apiVersion: apps/v1beta1 kind: Deployment metadata: name: nginx spec: selector: matchLabels: app: nginx template: metadata: labels: app: nginx spec: containers: - image: nginx name: mysql ports: - containerPort: 80 volumeMounts: - name: nginx-persistent-storage mountPath: /usr/share/nginx/html volumes: - name: nginx-persistent-storage persistentVolumeClaim: claimName: nginx-pvc ``` > (3)根据yaml文件创建资源并查看资源 ```shell kubectl apply -f nginx-pv-demo.yaml kubectl get pv,pvc kubectl get pods -o wide ``` > (4)测试持久化存储 ```shell 01 在/nfs/data/nginx新建文件1.html，写上内容 02 kubectl get pods -o wide 得到nginx-pod的ip地址 03 curl nginx-pod-ip/1.html 04 kubectl exec -it nginx-pod bash 进入/usr/share/nginx/html目录查看 05 kubectl delete pod nginx-pod 06 查看新nginx-pod的ip并且访问nginx-pod-ip/1.html ``` ###


StorageClass
官网：https://kubernetes.io/docs/concepts/storage/storage-classes/

nfs github：github：https://github.com/kubernetes-incubator/external-storage/tree/master/nfs
A StorageClass provides a way for administrators to describe the “classes” of storage they offer. Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators. Kubernetes itself is unopinionated about what classes represent. This concept is sometimes called “profiles” in other storage systems.
Each StorageClass contains the fields provisioner, parameters, and reclaimPolicy, which are used when a PersistentVolume belonging to the class needs to be dynamically provisioned.

The name of a StorageClass object is significant, and is how users can request a particular class. Administrators set the name and other parameters of a class when first creating StorageClass objects, and the objects cannot be updated once they are created.
StorageClass声明存储插件，用于自动创建PV。

说白了就是创建PV的模板，其中有两个重要部分：PV属性和创建此PV所需要的插件。

这样PVC就可以按“Class”来匹配PV。

可以为PV指定storageClassName属性，标识PV归属于哪一个Class。

\```yaml apiVersion: storage.k8s.io/v1 kind: StorageClass metadata: name: standard provisioner: kubernetes.io/aws-ebs parameters: type: gp2 reclaimPolicy: Retain allowVolumeExpansion: true mountOptions: - debug volumeBindingMode: Immediate ```
实战

\### StorageClass实战 > `github`： > > 网盘中:课堂源码/storage/有好几个yaml文件 > (1)准备好NFS服务器[并且确保nfs可以正常工作]，创建持久化需要的目录 > > path: /nfs/data/jack > > server: 121.41.10.13 > > 比如mkdir -p /nfs/data/jack > (2)根据rbac.yaml文件创建资源 ```shell kubectl apply -f rbac.yaml ``` > (3)根据deployment.yaml文件创建资源 ```shell kubectl apply -f deployment.yaml ``` > (4)根据class.yaml创建资源 ```shell kubectl apply -f class.yaml ``` > (5)根据pvc.yaml创建资源 ```shell kubectl apply -f my-pvc.yaml kubectl get pvc ``` > (6)根据nginx-pod创建资源 ```shell kubectl apply -f nginx-pod.yaml kubectl exec -it nginx bash cd /usr/jack # 进行同步数据测试 ```


Secret


定义概念
Kubernetes secret objects let you store and manage sensitive information, such as passwords, OAuth tokens, and ssh keys.


Service Accout

\> 用于被 serviceaccount 引用。 > > serviceaccout 创建时 Kubernetes 会默认创建对应的 secret。Pod 如果使用了 serviceaccount，对应的 secret 会自动挂载到 Pod 的 /run/secrets/kubernetes.io/serviceaccount 目录中。 ```shell kubectl get secret # 可以看到service-account-token kubectl run nginx --image nginx kubectl get pods kubectl exec -it nginx-pod-name bash ls /run/secrets/kubernetes.io/serviceaccount ``` ```shell kubectl get secret kubectl get pods pod-name -o yaml # 找到volumes选项，定位到-name，secretName # 找到volumeMounts选项，定位到mountPath: /var/run/secrets/kubernetes.io/serviceaccount ```
kubernetes.io/service-account-token：用于被 serviceaccount 引用。serviceaccout 创建时 Kubernetes 会默认创建对应的 secret。Pod 如果使用了 serviceaccount，对应的 secret 会自动挂载到 Pod 的 /run/secrets/kubernetes.io/serviceaccount 目录中。


Opaque Secret
Opaque：使用base64编码存储信息，可以通过`base64 --decode`解码获得原始数据，因此安全性弱。


特殊说明
Opaque类型的Secret的value为base64位编码后的值


创建
从文件中创建

\``` echo -n "admin" > ./username.txt echo -n "1f2d1e2e67df" > ./password.txt ``` ```shell kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt ``` ``` kubectl get secret ```
使用yaml文件创建

\> (1)对数据进行64位编码 ```shell echo -n 'admin' | base64 echo -n '1f2d1e2e67df' | base64 ``` > (2)定义mysecret.yaml文件 ```yaml apiVersion: v1 kind: Secret metadata: name: mysecret type: Opaque data: username: YWRtaW4= password: MWYyZDFlMmU2N2Rm ``` > (3)根据yaml文件创建资源并查看 ```shell kubectl create -f ./secret.yaml kubectl get secret kubectl get secret mysecret -o yaml ```


使用
Secret挂载到Volume

\> kubectl apply -f mypod.yaml ```yaml apiVersion: v1 kind: Pod metadata: name: mypod spec: containers: - name: mypod image: redis volumeMounts: - name: foo mountPath: "/etc/foo" readOnly: true volumes: - name: foo secret: secretName: mysecret ``` ```shell kubectl exec -it pod-name bash ls /etc/foo cat /etc/foo/username cat /etc/foo/password ```
Secret导出到环境变量之中

\```yaml apiVersion: v1 kind: Pod metadata: name: secret-env-pod spec: containers: - name: mycontainer image: redis env: - name: SECRET_USERNAME valueFrom: secretKeyRef: name: mysecret key: username     - name: SECRET_PASSWORD valueFrom: secretKeyRef: name: mysecret key: password restartPolicy: Never ```


kubernetes.io/dockerconfigjson
kubernetes.io/dockerconfigjson：用于存储docker registry的认证信息。
可以直接使用`kubectl create secret`命令创建
无论是ConfigMap，Secret，还是DownwardAPI，都是通过ProjectedVolume实现的，可以通过APIServer将信息放到Pod中进行使用。


ConfigMap


定义概念
ConfigMaps allow you to decouple configuration artifacts from image content to keep containerized applications portable.
用来保存配置数据的键值对，也可以保存单个属性，也可以保存配置文件。

所有的配置内容都存储在etcd中，创建的数据可以供Pod使用。


创建configMap
使用目录创建

\#### 从目录中创建 ```shell mkdir config cd config mkdir a mkdir b cd .. ``` ``` kubectl create configmap config --from-file=config/ kubectl get configmap ```
使用命令创建

\#### 命令行创建 ```shell # 创建一个名称为my-config的ConfigMap，key值时db.port，value值是'3306' kubectl create configmap my-config --from-literal=db.port='3306' kubectl get configmap ``` > 详情信息：kubectl get configmap myconfig -o yaml > > ```yaml > apiVersion: v1 > data: > db.port: "3306" > kind: ConfigMap > metadata: > creationTimestamp: "2019-11-22T09:50:17Z" > name: my-config > namespace: default > resourceVersion: "691934" > selfLink: /api/v1/namespaces/default/configmaps/my-config > uid: 7d4f338b-0d0d-11ea-bb46-00163e0edcbd > ```


使用文件创建
从配置文件中创建

从配置文件中创建 创建一个文件，名称为app.properties ```properties name=jack age=17 ``` ```shell kubectl create configmap app --from-file=./app.properties kubectl get configmap kubectl get configmap app -o yaml ```
通过yaml文件创建

configmaps.yaml ```YAML apiVersion: v1 kind: ConfigMap metadata: name: special-config namespace: default data: special.how: very --- apiVersion: v1 kind: ConfigMap metadata: name: env-config namespace: default data: log_level: INFO ``` kubectl apply -f configmaps.yaml kubectl get configmap


Pod中使用configMap


ConfigMap代替环境变量
通过环境变量的方式，直接传递给pod

\> 使用valueFrom、configMapKeyRef、name > > key的话指定要用到的key > > test-pod.yaml > > kubectl logs pod-name ```yaml apiVersion: v1 kind: Pod metadata: name: dapi-test-pod spec: containers: - name: test-container image: busybox command: [ "/bin/sh", "-c", "env" ] env: # Define the environment variable - name: SPECIAL_LEVEL_KEY valueFrom: configMapKeyRef: # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY name: special-config # Specify the key associated with the value key: special.how restartPolicy: Never ```


ConfigMap设置命令行参数
通过在pod的命令行下运行的方式(启动命令中)

\> 在命令行下引用时，需要先设置为环境变量，之后可以用过$(VAR_NAME)设置容器启动命令的启动参数 > > test-pod2.yaml > > kubectl logs pod-name ```yaml apiVersion: v1 kind: Pod metadata: name: dapi-test-pod2 spec: containers: - name: test-container image: busybox command: [ "/bin/sh", "-c", "echo $(SPECIAL_LEVEL_KEY)" ] env: - name: SPECIAL_LEVEL_KEY valueFrom: configMapKeyRef: name: special-config key: special.how restartPolicy: Never ```


通过数据卷插件使用ConfigMap
作为volume的方式挂载到pod内

\> 将创建的ConfigMap直接挂载至Pod的/etc/config目录下，其中每一个key-value键值对都会生成一个文件，key为文件名，value为内容。 > > kubectl apply -f pod-myconfigmap-v2.yml > > kubectl exec -it pod-name bash > > kubectl logs pod-name ```yaml apiVersion: v1 kind: Pod metadata: name: pod-configmap2 spec: containers: - name: test-container image: busybox command: [ "/bin/sh", "-c", "ls /etc/config/" ] volumeMounts: - name: config-volume mountPath: /etc/config volumes: - name: config-volume configMap: name: special-config restartPolicy: Never ```
注意：Pod只能使用同一个命名空间的ConfigMap


configMap热更新
实现演示

\> 在之前ingress网络中的mandatory.yaml文件中使用了ConfigMap，于是我们可以打开 ```YAML apiVersion: v1 kind: Namespace metadata: name: ingress-nginx labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx --- kind: ConfigMap apiVersion: v1 metadata: name: nginx-configuration namespace: ingress-nginx labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx --- kind: ConfigMap apiVersion: v1 metadata: name: tcp-services namespace: ingress-nginx labels:   app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx --- kind: ConfigMap apiVersion: v1 metadata: name: udp-services namespace: ingress-nginx labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx --- apiVersion: v1 kind: ServiceAccount metadata: name: nginx-ingress-serviceaccount namespace: ingress-nginx labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx --- apiVersion: rbac.authorization.k8s.io/v1beta1 kind: ClusterRole metadata: name: nginx-ingress-clusterrole labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx rules: - apiGroups: - "" resources: - configmaps - endpoints - nodes - pods - secrets verbs: - list - watch - apiGroups: - "" resources: - nodes verbs: - get - apiGroups: - "" resources: - services verbs: - get - list - watch - apiGroups: - "" resources: - events verbs: - create - patch - apiGroups: - "extensions" - "networking.k8s.io" resources: - ingresses verbs: - get - list - watch - apiGroups: - "extensions" - "networking.k8s.io" resources: - ingresses/status verbs: - update --- apiVersion: rbac.authorization.k8s.io/v1beta1 kind: Role metadata: name: nginx-ingress-role namespace: ingress-nginx labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx rules: - apiGroups: - "" resources: - configmaps - pods - secrets - namespaces verbs: - get - apiGroups: - "" resources: - configmaps resourceNames: # Defaults to "-" # Here: "-" # This has to be adapted if you change either parameter # when launching the nginx-ingress-controller. - "ingress-controller-leader-nginx" verbs: - get - update - apiGroups: - "" resources: - configmaps verbs: - create - apiGroups: - "" resources: - endpoints verbs: - get --- apiVersion: rbac.authorization.k8s.io/v1beta1 kind: RoleBinding metadata: name: nginx-ingress-role-nisa-binding namespace: ingress-nginx labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx roleRef: apiGroup: rbac.authorization.k8s.io kind: Role name: nginx-ingress-role subjects: - kind: ServiceAccount name: nginx-ingress-serviceaccount namespace: ingress-nginx --- apiVersion: rbac.authorization.k8s.io/v1beta1 kind: ClusterRoleBinding metadata: name: nginx-ingress-clusterrole-nisa-binding labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx roleRef: apiGroup: rbac.authorization.k8s.io kind: ClusterRole name: nginx-ingress-clusterrole subjects: - kind: ServiceAccount name: nginx-ingress-serviceaccount namespace: ingress-nginx --- apiVersion: apps/v1 kind: Deployment metadata:  name: nginx-ingress-controller namespace: ingress-nginx labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx spec: replicas: 1 selector: matchLabels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx template: metadata: labels: app.kubernetes.io/name: ingress-nginx app.kubernetes.io/part-of: ingress-nginx annotations: prometheus.io/port: "10254" prometheus.io/scrape: "true" spec: # wait up to five minutes for the drain of connections terminationGracePeriodSeconds: 300 serviceAccountName: nginx-ingress-serviceaccount hostNetwork: true #设置为hostPort nodeSelector: name: ingress kubernetes.io/os: linux containers: - name: nginx-ingress-controller image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.26.1 args: - /nginx-ingress-controller - --configmap=$(POD_NAMESPACE)/nginx-configuration - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services - --udp-services-configmap=$(POD_NAMESPACE)/udp-services - --publish-service=$(POD_NAMESPACE)/ingress-nginx - --annotations-prefix=nginx.ingress.kubernetes.io securityContext: allowPrivilegeEscalation: true capabilities: drop: - ALL add: - NET_BIND_SERVICE # www-data -> 33 runAsUser: 33 env: - name: POD_NAME valueFrom: fieldRef: fieldPath: metadata.name - name: POD_NAMESPACE valueFrom: fieldRef: fieldPath: metadata.namespace ports: - name: http containerPort: 80 - name: https containerPort: 443 livenessProbe: failureThreshold: 3 httpGet: path: /healthz port: 10254 scheme: HTTP initialDelaySeconds: 10 periodSeconds: 10 successThreshold: 1 timeoutSeconds: 10 readinessProbe: failureThreshold: 3 httpGet: path: /healthz port: 10254 scheme: HTTP periodSeconds: 10 successThreshold: 1 timeoutSeconds: 10 lifecycle: preStop: exec: command: - /wait-shutdown --- ``` > > 可以发现有nginx-configuration、tcp-services等名称的cm > > 而且也可以发现最后在容器的参数中使用了这些cm > > ```yaml > containers: > - name: nginx-ingress-controller > image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.26.1 > args: > - /nginx-ingress-controller > - --configmap=$(POD_NAMESPACE)/nginx-configuration > - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services > - --udp-services-configmap=$(POD_NAMESPACE)/udp-services > - --publish-service=$(POD_NAMESPACE)/ingress-nginx > - --annotations-prefix=nginx.ingress.kubernetes.io > ``` > > 开启证明之旅和cm的使用方式 > (1)查看nginx ingress controller的pod部署 > > kubectl get pods -n ingress-nginx -o wide ``` NAME READY STATUS RESTARTS AGE nginx-ingress-controller-7c66dcdd6c-v8grg 1/1 Running 0 8d NAME READY STATUS RESTARTS AGE IP NODE NOMINATED NODE READINESS GATES nginx-ingress-controller-7c66dcdd6c-v8grg 1/1 Running 0 8d 172.16.31.150 w1 ``` > (2)发现运行在w1节点上，说明w1上一定有对应的container，来到w1节点 > > docker ps | grep ingress ``` ddde4b354852 quay.io/kubernetes-ingress-controller/nginx-ingress-controller "/usr/bin/dumb-init …" 8 days ago Up 8 days k8s_nginx-ingress-controller_nginx-ingress-controller-7c66dcdd6c-v8grg_ingress-nginx_b3e2f9a5-0943-11ea-b2b3-00163e0edcbd_0 b6b7412855c5 k8s.gcr.io/pause:3.1 "/pause" 8 days ago Up 8 days k8s_POD_nginx-ingress-controller-7c66dcdd6c-v8grg_ingress-nginx_b3e2f9a5-0943-11ea-b2b3-00163e0edcbd_0 ``` > (3)不妨进入容器看看？ > > docker exec -it ddde4b354852 bash > (4)可以发现，就是一个nginx嘛，而且里面还有nginx.conf文件，美滋滋 ``` /etc/nginx/nginx.conf ``` > (5)不妨打开nginx.conf文件看看 > > 假如已经配置过ingress，不妨尝试搜索一下"k8s.demoxxx"/"itcrazy2016.com" ```conf server { server_name k8s.itcrazy2016.com ; ``` > (6)到这里，大家应该有点感觉了，原来nginx ingress controller就是一个nginx，而所谓的ingress.yaml文件中配置的内容像itcrazy2016.com就会对应到nginx.conf中。 > (7)但是，不可能每次都进入到容器里面来修改，而且还需要手动重启nginx，很麻烦 > > 一定会有好事之者来做这件事情，比如在K8s中有对应的方式，修改了什么就能修改nginx.conf文件 > (8)先查看一下nginx.conf文件中的内容，比如找个属性：proxy_connect_timeout 5s > > 我们想要将这个属性在K8s中修改成8s，可以吗？ > > kubectl get cm -n ingress-nginx > ```yaml kind: ConfigMap apiVersion: v1 metadata: name: nginx-configuration namespace: ingress-nginx  labels: app: ingress-nginx data: proxy-read-timeout: "208" ``` > > kubectl apply -f nginx-config.yaml > > kubectl get cm -n ingress-nginx > (9)再次查看nginx.conf文件 > (10)其实定义规则都在nginx ingress controller的官网中 > > > >
更新触发说明


## 调度器


调度器概念
概念
调度过程
自定义调度器


调度亲和性


nodeAffinity
preferredDuringSchedulingIgnoredDuringExecution
requiredDuringSchedulingIgnoredDuringExecution


podAntiAffinity
preferredDuringSchedulingIgnoredDuringExecution
requiredDuringSchedulingIgnoredDuringExecution
亲和性运算符


污点
污点概念


Taint
组成
污点的设置、查看和去除


Tolerations
tolerations 设置演示


固定节点调度
PodName 指定调度
标签选择器调度


## 集群安全机制
机制说明
准入控制


鉴权控制
AlwaysDeny
AlwaysAllow
ABAC
Webbook


RBAC
Role and ClusterRole
RoleBinding and ClusterRoleBinding
Resources
to Subjects
创建一个系统用户管理 k8s dev 名称空间：重要实验


认证
HTTP Token
HTTP Base
HTTPS


## HELM


HELM 概念


HELM 概念说明
Helm是Kubernetes的软件包管理工具，类似于Ubuntu中的apt、CentOS中的yum等。 
可以快速查找、下载和安装软件包，Helm由客户端组件helm和服务端组件tiller组成。


组将构成


chart 
helm的打包格式叫chart，chart即一系列文件，描述了一组相关的k8s集群资源


helm 
客户端命令行工具，用于本地开发及管理chart、chart仓库等


tiller 
helm的服务端，tiller接收helm的请求，与k8s的apiserver打交道，根据chart生成一个release并且管 理release


release
helm install命令在k8s集群中部署的chart称为release 


repository helm chart
helm客户端通过http协议来访问存储库中chart的索引文件和压缩包
HELM 部署
HELM 自定义


HELM 部署实例
HELM 部署 dashboard


metrics-server
HPA 演示


资源限制
Pod
名称空间
Prometheus
ELK


## 运维
高可用集群
修改源码
指定Pod所运行的Node

\> (1)给node打上label ``` kubectl get nodes kubectl label nodes worker02-kubeadm-k8s name=jack ``` > (2)查看node是否有上述label ``` kubectl describe node worker02-kubeadm-k8s ``` > (3)部署一个mysql的pod > > vi mysql-pod.yaml ```yaml apiVersion: v1 kind: ReplicationController metadata: name: mysql-rc labels: name: mysql-rc spec: replicas: 1 selector: name: mysql-pod template: metadata: labels: name: mysql-pod spec: nodeSelector: #关键点 name: jack containers: - name: mysql image: mysql imagePullPolicy: IfNotPresent ports: - containerPort: 3306 env: - name: MYSQL_ROOT_PASSWORD value: "mysql" --- apiVersion: v1 kind: Service metadata: name: mysql-svc labels: name: mysql-svc spec: type: NodePort ports: - port: 3306 protocol: TCP targetPort: 3306 name: http nodePort: 32306  selector: name: mysql-pod ``` > (4)查看pod运行详情 ``` kubectl apply -f mysql-pod.yaml kubectl get pods -o wide ```


kebectl命令总结
显示Pod的更多信息

kubectl get pod <pod-name> -o wide

以yaml格式显示Pod的详细信息

kubectl get pod <pod-name> -o yaml

查看pod的label标签：

kubectl get pods --show-labels

查看namespaces
kubectl get pods -n <namespaces>
查看一下当前的命名空间：kubectl get namespaces


\1. 创建资源对象

根据yaml配置文件一次性创建service和rc

kubectl create -f my-service.yaml -f my-rc.yaml

根据<directory>目录下所有.yaml、.yml、.json文件的定义进行创建操作

kubectl create -f <directory>
\2. 查看资源对象

查看所有Pod列表

kubectl get pods

查看rc和service列表

kubectl get rc,service(svc)
\3. 描述资源对象

显示Node的详细信息

kubectl describe nodes <node-name>

显示Pod的详细信息

kubectl describe pods/<pod-name>

显示由RC管理的Pod的信息

kubectl describe pods <rc-name>
\4. 删除资源对象

基于Pod.yaml定义的名称删除Pod

kubectl delete -f pod.yaml

删除所有包含某个label的Pod和service

kubectl delete pods,services -l name=<label-name>

删除所有Pod

kubectl delete pods --all
\5. 执行容器的命令

执行Pod的data命令，默认是用Pod中的第一个容器执行

kubectl exec <pod-name> data

指定Pod中某个容器执行data命令

kubectl exec <pod-name> -c <container-name> data

通过bash获得Pod中某个容器的TTY，相当于登录容器

kubectl exec -it <pod-name> -c <container-name> bash
6.Pod的扩容与缩容

执行扩容缩容Pod的操作

kubectl scale rc redis --replicas=3

我们需要确认的是在rc配置文件中定义的replicas数量，当我们执行上述命令的结果大于replicas的数量时，则我们执行的命令相当于扩容操作，反之相反，可以理解为我们填写的数量是我们需要的Pod数量。需要注意的是，当我们需要进行永久性扩容时，不要忘记修改rc配置文件中的replicas数量。
7.Pod的滚动升级

执行滚动升级操作

kubectl rolling-update redis -f redis-rc.update.yaml

需要注意的是当我们执行rolling-update命令前需要准备好新的RC配置文件以及ConfigMap配置文件，RC配置文件中需要指定升级后需要使用的镜像名称，或者可以使用kubeclt rolling-update redis --image=redis-2.0直接指定镜像名称的方式直接升级。


yaml文件配置的
pod
container
label
service
namespace
selector
replicaset
replicaController
deployment
statefulSet
configMap/secret
job
network
daemonSet
storage
lifeCycle
resource
部署 spring boot项目

\## 部署Spring Boot项目 > `流程`：确定服务-->编写Dockerfile制作镜像-->上传镜像到仓库-->编写K8S文件-->创建 > > `网盘/Kubernetes实战走起/课堂源码/springboot-demo` > (1)准备Spring Boot项目springboot-demo ```java @RestController public class K8SController { @RequestMapping("/k8s") public String k8s(){ return "hello K8s!"; } } ``` > (2)生成xxx.jar，并且上传到springboot-demo目录 ``` mvn clean pakcage ``` > (3)编写Dockerfile文件 > > mkdir springboot-demo > > cd springboot-demo > > vi Dockerfile ```dockerfile FROM openjdk:8-jre-alpine COPY springboot-demo-0.0.1-SNAPSHOT.jar /springboot-demo.jar ENTRYPOINT ["java","-jar","/springboot-demo.jar"] ``` > (4)根据Dockerfile创建image ```xshell docker build -t springboot-demo-image . ``` > (5)使用docker run创建container ``` docker run -d --name s1 springboot-demo-image ``` > (6)访问测试 ``` docker inspect s1 curl ip:8080/k8s ``` > (7)将镜像推送到镜像仓库 ```shell # 登录阿里云镜像仓库 docker login --username=itcrazy2016@163.com registry.cn-hangzhou.aliyuncs.com docker tag springboot-demo-image registry.cn-hangzhou.aliyuncs.com/itcrazy2016/springboot-demo-image:v1.0 docker push registry.cn-hangzhou.aliyuncs.com/itcrazy2016/springboot-demo-image:v1.0 ``` > (8)编写Kubernetes配置文件 > > vi springboot-demo.yaml > > kubectl apply -f springboot-demo.yaml ```yaml # 以Deployment部署Pod apiVersion: apps/v1 kind: Deployment metadata: name: springboot-demo spec: selector: matchLabels: app: springboot-demo replicas: 1 template: metadata: labels: app: springboot-demo spec: containers: - name: springboot-demo image: registry.cn-hangzhou.aliyuncs.com/itcrazy2016/springboot-demo-image:v1.0 ports: - containerPort: 8080 --- # 创建Pod的Service apiVersion: v1 kind: Service metadata: name: springboot-demo spec: ports: - port: 80 protocol: TCP targetPort: 8080 selector: app: springboot-demo --- # 创建Ingress，定义访问规则，一定要记得提前创建好nginx ingress controller apiVersion: extensions/v1beta1 kind: Ingress metadata: name: springboot-demo spec: rules: - host: k8s.demo.gper.club http: paths: - path: / backend: serviceName: springboot-demo servicePort: 80 ``` > (9)查看资源 ``` kubectl get pods kubectl get pods -o wide curl pod_id:8080/k8s kubectl get svc kubectl scale deploy springboot-demo --replicas=5 ``` > (10)win配置hosts文件[一定要记得提前创建好nginx ingress controller] ``` 192.168.0.61 springboot.jack.com ``` > (11)win浏览器访问 ``` http://springboot.jack.com/k8s ``` ##