# 概念

![image-20211019145719697](C:\Users\liaosl\OneDrive\笔记\Kubernetes\image-20211019145719697.png)

在 kubernetes 里面，Pod 实际上正是 kubernetes 项目抽象出来的一个可以类比为进程组的概念

  例如：现在有一个应用，它有四个职责不同、相互协作的进程，需要放在容器里去运行，在 Kubernetes 里面会把四个独立的进程分别用四个独立的容器启动起来，然后把它们定义在一个 Pod 里面。

  Pod 在 Kubernetes 里是一个逻辑单位，没有一个真实的东西可以对应；真正在物理上存在的是容器，多个容器的组合就叫做 Pod。

  Pod 是 Kubernetes 分配资源的一个单位，因为里面的容器要共享某些资源，所以 Pod 也是 Kubernetes 的原子调度单位。

 

亲密关系 - 调度解决

- 两个应用需要运行在同一台宿主机

超亲密关系 - Pod解决

- 会发生直接的文件交换
- 使用localhost或者Socket文件进行本地通信
- 会发生非常频繁的RPC调用
- 会共享某些Linux Namespace

 

Pod要解决的问题：如何让一个Pod里的多个容器之间最高效的共享某些资源和数据，即打破Linux Namespace和Cgroups对容器的隔离。

分为两部分：网络和存储：

1）共享网络：

- Kubernetes会在每个Pod里，额外创建一个Infra container小容器（非常小，由汇编语言编写，永远处于“暂停”状态）来共享整个Pod的Network Namespace；
- 其他容器会通过Join Namespace的方式加入到这个Network Namespace中；
- 其他容器看到的网络视图是完全一样的，都来自于Infra container；
- 一个Pod只有一个IP地址，就是这个Infra container的IP地址；
- 在 Pod 里，Infra container是第一个启动，并且整个 Pod 的生命周期是等同于 Infra container 的生命周期的，与其他容器无关；
- 在 Kubernetes 里面，允许去单独更新 Pod 里的某一个镜像，即：做这个操作，整个 Pod 不会重建，也不会重启。

2）共享存储：

  将volume设置为Pod level

 

 

容器设计模式：Sidecar

  在 Pod 里面，可以定义一些专门的容器，来执行主业务容器所需要的一些辅助工作



# 基本概念

1）可以在kubernetes中创建和管理的最小可部署计算单元；

2）是多个（或一个）容器的组合，提供两种共享资源：存储和网络，使用共同的运行容器的规范；

3）Pod的内容是共同定位和共同调度的，并在共享上下文（一组Linux名称空间、cgroups和潜在的其他隔离）中运行；

4）Pod构建了一下“逻辑主机”模型：它包含一个或多个相对紧密耦合的应用程序容器；

5）Pod可以在启动期间运行一个init容器，该容器和其他应用程序容器一样；

6）一般来说，不会直接创建pod（即使是单例pod），而是使用Deployment或Job等工作负载资源创建它们；如果pod需要跟踪状态，可以使用StatefulSet。

7）Kubernetes管理Pod，不会直接管理容器；

8）Pod中的容器会被放置在集群中的同一个物理机或虚拟机中；

9）在kubernetes中很少创建单独的pod（即使是单例pod）。这是因为pod被设计成相对短暂的、一次性的实体。当Pod被创建时，新的Pod将被调度在集群中的节点上运行，会一直在该节点上，直到Pod执行完成、Pod对象被删除、Pod资源不足被逐出或节点故障。

10）在Pod中重新启动容器和重新启动Pod是不一样的。Pod不是一个进程，而是一个运行容器的环境。Pod一直存在，直到被删除。

11）您可以使用工作负载资源为您创建和管理多个pod。资源的控制器处理复制、rollout和Pod故障时的自动恢复。例如，如果一个Node发生故障，控制器会注意到该Node上的Pod已经停止工作，并创建一个替换Pod。调度程序将替换Pod放置到运行正常的节点上。

# 静态Pod