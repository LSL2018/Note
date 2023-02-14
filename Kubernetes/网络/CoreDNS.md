# 基本概念

1）CoreDNS灵活，可扩展，可以作为K8S集群DNS；

2）可以在集群中使用CoreDNS而不是kube-dns，方法是替换现有deployment中的kube-dns，或者使用kubeadm之类的工具部署和升级集群。





# CoreDNS和kube-dns



# 安装CoreDNS





# 迁移到CoreDNS

​	在Kubernetes 1.10及更高版本中，当使用kubeadm升级使用kube-dns的集群时，可以移动到CoreDNS。在这种情况下，kubeadm将生成基于kube-dns ConfigMap的CoreDNS配置(“Corefile”)，保留stub域和上游名称服务器的配置。

​	如果从kube-dns转移到CoreDNS，要确保在升级期间将CoreDNS功能门设置为true。例如，以下是v1.11.0升级的样子:

```shell
kubeadm upgrade apply v1.11.0 --feature-gates=CoreDNS=true
```

​	在Kubernetes 1.13及更高版本中，CoreDNS功能门被移除，CoreDNS被默认使用。

​	在1.11之前的版本中，Corefile会被升级过程中创建的文件覆盖。如果您已经定制了ConfigMap，那么您应该保存现有的ConfigMap。您可以在新的ConfigMap启动并运行后重新应用您的自定义。

​	如果你在Kubernetes 1.11及更高版本运行CoreDNS，在升级期间，你现有的Corefile将被保留。

​	在Kubernetes 1.21版本中，从kubeadm中删除了对kube-dns的支持。



# 升级CoreDNS

1）在升级之前，识别并解析Corefile中与新的CoreDNS版本向后不兼容的任何配置。









