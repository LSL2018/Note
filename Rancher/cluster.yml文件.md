# 节点选项

## Address

- 设置节点的主机名或ip地址
- rke必须能够连接到这个地址

## internal address

- 没有设置internal_address，就使用address进行主机间通信
- 设置用于k8s组件的主机间通信的地址
- 提供了让具有多个地址的节点设置一个特定的地址用于私有网络上的主机间通信的能力

## hostname_override

- 提供一个友好的名称，在注册节点时使用
- 不需要是可路由地址，但必须是一个有效的k8s资源名
- 没有设置的话，在kubernetes注册节点时就会使用address指令

## ssh port

- 指定连接到这个节点时要使用的端口，默认端口是22

## ssh users

- 指定连接到这个节点时要使用的user，该用户必须是docker组的用户，或者允许向节点的Docker套接字写入

## ssh key path

- 用于连接到这个节点时要使用的 ssh 私钥路径
- 默认密钥路径：~/.ssh/id_rsa

## ssh key

- 不设置ssh密钥的路径，指定实际的密钥

## docker socker

- 默认：/var/run/docker.sock

## labels

- 添加标签映射
- 使用入口控制器的node_selector选项时，可以使用

## taints

- 为每个节点添加污点



# 集群层级选项

## cluster_name：

- 默认情况下，cluster_name是local

## ignore_docker_version：

- 默认情况下，rke会检查所有主机上已安装的docker的版本，如果Kubernetes不支持该版本的docker，会导致rke运行失败并且报错
- 是一个 boolean 类型的参数，表示在运行 RKE 前是否执行 Docker 版本检测，默认值为`false`

## 



## 