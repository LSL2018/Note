# 综述

## 什么是service

- 将运行在一组pod上的应用程序作为网络服务公开的一种抽象方法

- 为一组功能相同的Pod提供单一不变的接入口；当服务存在时，其IP地址和端口不会改变。客户端通过 IP 地址和端口号建立连接，这些连接会被路由到提供该服务的任意一个 pod 上。这样客户端就不需要知道每个单独的提供服务的 pod 的地址，这些 pod 就可以在集群中随时被创建或移除。

  示例：

<img src="C:\Users\liaosl\OneDrive\笔记\Kubernetes\网络\image-20211103112628421.png" alt="image-20211103112628421" style="zoom: 67%;" />

- Kuberenetes为Pods提供了私有的IP地址和一组Pods的单个DNS名称，并可以在它们之间进行负载平衡

## 使用动机

- Kubernetes Pods会根据集群的状态被创建和销毁。Pod是非永久资源。如果使用Deployment来运行应用程序，它可以动态地创建和销毁pod。
- 每个Pod都有自己的IP地址，但是在Deployment中，某一时刻运行的Pod集可能与稍后运行该应用程序的Pod集不同。这就导致了一个问题:如果一组pod(称为“后端”)为集群中的其他pod(称为“前端”)提供功能，那么前端如何找出并跟踪哪个IP地址呢



## Service资源

- 在Kubernetes中，服务是一种抽象，它定义了一组pod的逻辑集合和访问它们的策略(有时这种模式被称为微服务)。
- 服务的目标Pods集合通常由选择器决定。
- 如果能够在应用程序中使用Kubernetes API来发现服务，那么可以在API服务器中查询endpoints，这些endpoint在服务中的pod集发生变化时就会更新。对于非本机应用程序，Kubernetes提供了在应用程序和后端pod之间放置网络端口或负载平衡器的方法。



# 定义服务

## 综述

- 在Kubernetes里，服务是REST对象，类似于Pod。
- 和所有REST对象一样，可以将一个服务定义POST到API服务器，以创建新实例。
- 服务对象的名字必须是有效的RFC 1035标签名。



## 创建服务

### 有选择器

​	假设有一组Pod，其中每个pod侦听TCP端口9376，并包含一个标签app=MyApp：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

- 上述规范创建一个名为“my-service”的新服务对象，以任何带有app=MyApp标签的Pod上的TCPD端口9376为目标。
- kubernetes为该服务分配一个IP地址（有时称为“cluster IP”），由服务代理使用。
- Service选择器的控制器不断扫描与它的选择器匹配的Pod，然后将任何更新发布到同样名为“my-service”的端点对象。
- 服务可以将任何传入端口映射到targetPort，默认情况下targetPort被设置为和port一样。
- 默认协议是TCP。
- Pod里的端口定义是有名字的，可以在targetPort属性中引用。
- 由于许多服务需要公开多个端口，Kubernetes支持服务对象上的多个端口定义。每个端口定义可以有相同的协议，也可以有不同的协议。

### 没有选择器

#### 使用场景

​	服务通常抽象对Kubernetes Pods的访问，但也可以抽象其他类型的后端：

- 生产环境有外部数据库集群，但测试环境中使用自己的数据库
- 将服务指向另一个名称空间或另一个集群上的服务
- 将工作负载迁移到kubernetes，在评估时只在kubernetes中运行部分后端

#### 示例

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

- 此服务没有选择器，所以不会自动创建相应的endpoint对象。通过手动添加endpoint对象，可以手动将服务映射到它运行的网络地址和端口:

  ```yaml
  apiVersion: v1
  kind: Endpoints
  metadata:
    name: my-service
  subsets:
    - addresses:
        - ip: 192.0.2.42
      ports:
        - port: 9376
  ```

  - endpoint IPS不能是： 127.0.0.0/8 for IPv4, ::1/128 for IPv6； 169.254.0.0/16， 224.0.0.0/24 for IPv4, fe80::/64 for IPv6；其他Kubernetes服务的cluster IP

- 访问没有选择器的服务的工作原理与访问有选择器的服务的工作原理相同。

- Kubernetes API服务器不允许代理未映射到pod的endpoint。因此在没有选择器的服务中，类似于 kubectl proxy <service-name>的操作会失败

## EndpointSlices

- 如果endpoints资源里有超过1000个endpoints，kubernetes（v1.22或更高版本）会标记该endpoint：

  ```yaml
  endpoints.kubernetes.io/over-capacity: truncated
  ```

  - 该标记表表明受影响的endpoints对象过多，endpoints控制器会将endpoints的数量截断为1000

- 详情参考[EndpointSlices](./EndpointSlices.md)





# 虚拟IPS和服务代理

## 综述

- kubernetes集群的每个节点都运行一个kube-porxy，其为ExternalName类型以外的服务实现虚拟IP。
- 相比轮询DNS，为服务使用代理的原因有：
  - DNS有很长一段时间没有考虑到TTLs的记录，并且在名称查找结果过期后还缓存它们
  - 有些应用只做一次DNS查询，但会无限期缓存结果
  - 即使应用程序和库进行了适当的重新解析，DNS记录上的低或零TTLs会带来高负载，难以管理
- 在运行kube-proxy时，内核级别的规则可能被修改，在某些情况下，直到reboot后才会被清除。因此，运行kube-proxy只能由管理员来完成，其必须了解在计算机上使用低级别、特权网络代理服务的后果。尽管kube-proxy可执行文件支持一个cleanup函数，但该函数不是官方特性，因此仅可用。

## 配置

​	kube-proxy可以根据不同的配置，以不同的模式启动：

- kube-proxy的配置是通过ConfigMap完成，其ConfigMap不支持实时重载配置
- kube-proxy的ConfigMap参数不能在启动时全部进行验证



## 代理模式

- 在代理模式中，服务的IP:Port绑定的流量被代理到适当的后端，而客户机不知道关于Kubernetes、Services或Pods的任何信息。
- 设置service.spec.sessionAffinity为“ClientIP”（默认为None），可以确保特定客户端的每次连接都传递到相同的Pod。
- 设置service.spec.sessionAffinityConfig.clientIP.timeoutSeconds，可以修改最大会话保持时间（默认10800，即3小时）。

### 用户空间代理模式

- kube-proxy将监视kubernetes控制平面，以查看服务和endpoint对象的添加和删除。
- 对每个服务，在本地节点上随机打开一个端口，任何到这个“代理端口”的连接都被代理到服务的一个后端（通过endpoint报告）。
- kube-porxy再决定使用哪个后端Pod时考虑服务的SessionAffinity设置。
- 用户空间代理安装iptables规则，这些规则捕获到服务的cluster IP（虚拟的）和端口的流量，并将其重定向到代理后端Pod的代理端口。
- 默认情况下，通过轮询算法选择后端。

<img src="C:\Users\liaosl\OneDrive\笔记\Kubernetes\网络\image-20211108111531724.png" alt="image-20211108111531724" style="zoom: 50%;" />



### iptables代理模式

- kube-proxy将监视kubernetes控制平面，以查看服务和endpoint对象的添加和删除。
- 对每个服务，安装iptables规则，这些规则捕获到服务的cluster IP（虚拟的）和端口的流量，并将其重定向到代理后端Pod的代理端口。
- 对每个endpoint对象，安装用于选择后端Pod的iptables规则。
- 默认情况下，随机选择后端。
- 使用iptables处理流量的系统开销较低，因为流量是由Linux netfilter处理的，不需要在用户空间和内核空间之间切换。
- 选择的第一个Pod没有响应，则连接失败，这和用户空间代理模式不同。
- 可以使用Pod readiness probes来验证后端Pod是否正常工作，以免将流量发送到已失败的Pod。

<img src="C:\Users\liaosl\OneDrive\笔记\Kubernetes\网络\image-20211108113721622.png" alt="image-20211108113721622" style="zoom:50%;" />







### IPVS代理模式

- 在v1.11版本后可用
- kube-proxy监控kubernetes的服务和endpoint，通过netlink接口创建IPVS规则，并定期同步服务和endpoint的IPVS规则；这样确保IPVS状态和期望状态匹配。
- 当访问一个服务时，IPVS将流量导向一个后端Pod。
- 基于netfilter hook函数，类似iptables模式，但使用哈希表作为底层数据结构，在内核空间工作。
- 提供更多选择来平衡后端pod的流量：
  - rr
  - lc
  - dh
  - sh
  - sed
  - nq
- 以IPVS模式运行kube-proxy，在启动kube-proxy之前，必须使节点上的IPVS可用。
- 以IPVS代理模式启动时，将验证IPVS内核模块是否可用。如果未检测到IPVS内核模块，kube-proxy将恢复为iptables代理模式运行。

<img src="C:\Users\liaosl\OneDrive\笔记\Kubernetes\网络\image-20211108140940664.png" alt="image-20211108140940664" style="zoom:50%;" />







# 多端口服务

- 对于某些服务，需要公开多个端口。

- Kubernetes允许在一个Service对象上配置多个端口定义。

- 当为一个服务使用多个端口时，必须给出所有端口的名称，以确保它们没有歧义。

- 示例：

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-service
  spec:
    selector:
      app: MyApp
    ports:
      - name: http
        protocol: TCP
        port: 80
        targetPort: 9376
      - name: https
        protocol: TCP
        port: 443
        targetPort: 9377
  ```

- 端口名称必须只包含小写字母数字字符和-，以字母数字字符开始和结束。





# 选择私有IP

- 可以在.spec.clusterIP参数指定私有的cluster IP作为服务创建请求的一部分
- 选择的IP必须是在API服务器配置的service-cluster-ip-range CIDR范围内





# 传输策略

## 外部传输

- 在v1.22版本及以后可用
- 设置.spec.externalTrafficPolicy来控制来自外部源的流量如何路由。
- 有“Cluster”和“Local”两种选择：
  - 前者是将外部流量路由到所有就绪的endpoint，后者是仅路由到就绪的本地节点的endpoint
- 为kube-proxy启用了ProxyTerminatingEndpoints特性门（feature gate），kube-proxy将检查节点是否具有本地endpoints以及是否将所有本地endpoints标记为终止：
  - 本地endpoint存在，并且都在终止：那么kube-proxy将忽略local的任何外部流量策略；



## 内部传输

- 在v1.22版本及以后可用
- 设置.spec.internalTrafficPolicy来控制来自内部源的流量如何路由。
- 有“Cluster”和“Local”两种选择：
  - 前者是将内部流量路由到所有就绪的endpoint，后者是仅路由到就绪的本地节点的endpoint。



# 服务发现

​	有两种方式查找一个服务：

- 环境变量
- DNS



## 环境变量

- 当Pod在节点上运行时，kubelet会为每个活跃的服务添加一组环境变量：支持Docker links compatible变量、{SVCNAME}_SERVICE_HOST变量和{SVCNAME}_SERVICE_POST变量

- 示例：服务redis-master（公开TCP端口:6379，已分配clusterIP:10.0.0.11）会生成以下环境变量

  ```sh
  REDIS_MASTER_SERVICE_HOST=10.0.0.11
  REDIS_MASTER_SERVICE_PORT=6379
  REDIS_MASTER_PORT=tcp://10.0.0.11:6379
  REDIS_MASTER_PORT_6379_TCP=tcp://10.0.0.11:6379
  REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
  REDIS_MASTER_PORT_6379_TCP_PORT=6379
  REDIS_MASTER_PORT_6379_TCP_ADDR=10.0.0.11
  ```

- 当有一个Pod需要访问一个服务，并且正在使用环境变量方法将端口和集群IP发布到客户机Pod时，必须在客户机Pod出现之前创建该服务。否则，这些客户机pod将无法填充它们的环境变量。（DNS服务发现就不需要）



## DNS

- 可以（或者说应该）使用插件为kubernetes集群设置DNS服务
- cluster-aware的DNS服务，比如CoreDNS，查看Kubernetes API中的新服务，并为每个服务创建一组DNS记录。
- 如果在整个集群中启用了DNS，那么所有pod都能够通过DNS名自动解析服务
- 可以为命名端口生成DNS SRV记录
- 访问ExternalName服务的唯一途径



# Headless服务

- 设置.spec.clusterIP为"None"，创建“headless”服务（不需要负载均衡和单一的服务IP）
- 可以和其他服务发现机制连接，不必和Kubernetes的实现捆绑在一起
- 不被分配cluster IP，kube-porxy不处理这些服务，没有负载均衡或代理，如何自动配置DNS取决于服务是否定义了选择器：
  - 有选择器： endpoints控制器在APIs创建一个Endpoints记录，并修改DNS配置以返回直接指向支持服务的Pod的A记录（IP地址）
  - 没有选择器：DNS系统寻找和配置：
    - ExternalName-type类型的服务的CNAME记录
    - 其他类型的服务的与服务名称相同的任何Endpoints记录





# 发布服务

## 综述

- 对于应用程序的某些部分(例如前端)，可能希望将服务公开到集群之外的外部IP地址。
- Kubernetes的“ServiceTypes”参数用以指定需要的服务类型，默认“ClusterIP”。
- 服务类型有：
  - ClusterIP
  - NodePort
  - LoadBalancer
  - ExternalName
- 还可以使用Ingress公开您的服务。Ingress不是服务类型，但可以充当集群的入口点，将路由规则合并到单个资源中，因为它可以在相同的IP地址下公开多个服务。





## NodePort

- Kubernetes控制平面从--service-node-port-range参数指定的范围（默认值：30000-32767）分配端口，该端口会被每个节点代理到服务中，并且服务会在.spec.ports[*].nodePort字段中记录

- 设置kube-porxy的--nodeport-address字段或kube-proxy配置文件的nodePortAddress字段，可以指定IP来代理端口

- 设置nodePort字段，指定端口号，但可能导致端口冲突

- 可以自由设置负载均衡解决方案

- 服务可以通过下列方式对外可见：

  ```
  <NodeIP>:spec.ports[*].nodePort
  .spec.clusterIP.spec.ports[*].port
  ```

- 示例：

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-service
  spec:
    type: NodePort
    selector:
      app: MyApp
    ports:
        # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
      - port: 80
        targetPort: 80
        # Optional field
        # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)
        nodePort: 30007
  ```

  





## LoadBalancer







## ExternalName

### 综述

- 将服务映射到一个DNS名称，而不是一个选择器

- 设置.spec.externalName参数，指定服务

- 示例：将prod命名空间的my-service服务映射到my.database.example.com

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-service
    namespace: prod
  spec:
    type: ExternalName
    externalName: my.database.example.com
  ```

  

- IPv4地址字符串，作为由数字组成的DNS名称，而不是IP地址；类似于IPv4地址的ExternalName不会被CoreDNS或ingress-nginx解析，因为这是指定一个规范的DNS名称

### External IPs

- 如果有外部IP路由到集群节点，kubernetes服务可以在这些外部IP上公开

- 示例：服务“my-service”可以被客户端在“80.11.12.10：80”上访问

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-service
  spec:
    selector:
      app: MyApp
    ports:
      - name: http
        protocol: TCP
        port: 80
        targetPort: 9376
    externalIPs:
      - 80.11.12.10
  ```



# 虚拟IP实现

## 避免碰撞

- kubernetes为每个服务分配私有IP，这样用户可以为自己的服务选择端口号，服务之间不会发生冲突
- 为了确保每个服务的IP唯一，内部分配器在创建服务前会自动更新在etcd里的全局分配映射表
- 服务必须在注册中心存在映射对象，才能获得IP地址分配，否则创建将失败，并显示无法分配IP地址的消息
- 在控制平面，后台控制器负责创建映射表，同时kubernetes控制器会检查不合理的分配，和清理已被分配但不被使用的IP



















