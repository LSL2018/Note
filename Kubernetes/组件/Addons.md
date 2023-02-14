# 概念

​	Addons使用Kubernetes资源(DaemonSet, Deployment等)来实现集群特性。因为它们提供了集群级别的特性，所以插件的命名空间资源属于kube系统命名空间。



# 组件

## DNS

1）集群的DNS是一个DNS服务器，为K8S服务提供DNS记录

 

## Web UI(Dashboard)

1）是Kubernetes集群的通用、基于web的UI。它允许用户管理在集群中运行的应用程序，以及集群本身，并排除故障

 

## Container Resource Monitoring

1）在中央数据库中记录有关容器的一般时间序列指标，并提供浏览该数据的UI。

 

## Cluster-level Logging

1）将容器日志保存到具有搜索/浏览界面的中央日志存储中