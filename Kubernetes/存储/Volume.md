# 综述

- Kubernetes支持多种类型的卷，Pod可以同时使用多种卷。

- 短暂卷（Ephemeral volume）的生命周期和Pod一样，而持久卷（Pesistent volume）会超过；当一个Pod停止时，短暂卷会被销毁，持久卷不会；对于给定Pod的任何类型的卷，在容器重新启动时都会保存数据。

- 卷的核心是一个目录，其中可能包含一些数据，可以被pod中的容器访问。

- 在.spec.volumes里指定要为Pod提供的卷，在.spec.containers[*].volumeMounts里声明卷在容器里的挂载位置。

- 卷不能挂载到其他卷或与其他卷有硬链接。

- Pod中每个容器必须独立指定挂载卷的位置。



# 卷的类型

## awsElasticBlockStore

### 综述



-  awsElasticBlockStore卷挂载一个Amazon Web Servies(AWS) EBS卷到Pod里；
- 删除Pod时，卷会卸载，但内容是持久化的，这意味着可以用数据预填充EBS卷，并且可以在Pod之间共享数据；
- 使用限制：
  - 运行Pod的节点必须是AWS EC2实例；
  - EBS只支持单个EC2实例挂载卷；
  - EC2实例需要位于与EBS卷相同的区域和可用性区域



### 创建卷





### CSI迁移







## azureDisk





## azureFile





## cephfs

### 综述

- cephfs卷允许将现有的CephFS卷挂载到Pod。
- Pod被删除时，卷会被卸载，内容会保留；cephfs卷可以预先填充数据，并且数据可以在pod之间共享。
- cephfs卷可以由多个写入器同时挂载。
- 在使用cephfs卷之前，必须使用导出的共享运行自己的Ceph服务器。





## cinder

### 综述

- Kubernetes必须配置OpenStack云提供商。
- cinder卷用于将OpenStack cinder卷挂载到pod中





