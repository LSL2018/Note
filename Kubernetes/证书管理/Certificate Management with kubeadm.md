# 综述

​	kubeadm生成的客户端证书有效期为1年





# 使用自定义证书

- 默认情况下，kubeadm可以生成集群需要的所有证书。
- 可以通过提供自定义的证书来覆盖：把证书放在由--cret-dir参数或kubeadm配置中的ClusterConfiguration指定的目录下，默认是/etc/kubernetes/pki。
- 如果在运行kubeadm init之前存在给定的证书和私钥对，kubeadm不会重写它们。这意味着可以将现有的CA复制到/etc/kubernetes/pki/ca.crt和/etc/kubernetes/pki/ca.key中，kubeadm将使用此CA对其余的证书进行签名



# 外部CA模式

​	只提供ca.crt文件，在其他证书和kubeconfig文件就绪的条件下，kubeadm会激活“外部CA”模式，在磁盘没有ca.key的情况下继续。

​	只适用于root CA。



# 检查证书是否过期

命令：

```shell
kubeadm certs check-expiration
```

- 该命令显示/ect/kubernetes/pki下的客户端证书和kubeadm使用的KUBECONFIG文件中嵌入的客户端证书的剩余时间
-  kubeadm 会通知用户证书是否由外部管理；在这种情况下，用户应该手动使用其他工具来管理证书更新
- kubeadm无法管理由外部CA签名的证书
- 该命令里不会有kubelet.conf，因为kubeadm将kubelet配置为自动更新证书，轮换的证书所在目录为/var/lib/kubelet/pki

# 自动更新证书

- kubeadm在升级控制平面时更新所有证书
- 如果对证书续订有更复杂的要求，可以通过将--certificate-renewal=false传递给kubeadm upgrade apply或kubeadm upgrade node来选择退出默认行为。





# 手动更新证书

​	可以使用下述命令手动更新证书，在任何时候：

```shell
kubeadm certs renew
```

- 该命令使用CA（或front-proxy-CA）证书和存储在/etc/kubernetes/pki中的密钥进行更新。
- 执行命令后，需要重启控制平面Pod，因为目前并不是所有组件和证书都支持动态重新加载证书
- 在HA集群中，该命令需要在所有控制平面节点上执行
- 静态pod由本地kubelet，而不是由API Server管理，因此不能使用kubectl来删除和重启它们。
  - 重新启动静态Pod:
    - 可以临时从/etc/kubernetes/manifest /中删除它的清单文件，然后等待20s（该数值由KubeletConfiguration里的fileCheckFrequency决定，默认20s）
    - kubelet会终止不在清单目录下的Pod
    - 将被删除的文件移回来，在下一个fileCheckFrequency周期之后，kubelet将重新创建Pod，这样就完成了组件的证书更新
- certs renew使用已有的证书作为属性（名字，组织，SAN等）的权威来源，而不是kubeadm-config ConfigMap。因此，最好让它们保持同步
- --csr-only参数可用于生成一个证书签名请求，使用外部CA更新证书（无需实际替换更新证书）
- 可以更新单个证书而不是全部证书



# 使用Kubernetes证书API更新证书

- 设置签名者（signer）：

  可以配置外部签名者，如cert-manager，或者使用内置签名者

  - 内置签名者是kube-controller-manager的一部分

  - 要激活内置签名者，需要添加--cluster-signing-cert-file和--cluster-signing-key-file参数。

    在创建新的集群时，可以在kubeadm的配置文件中添加：

    ```yaml
    apiVersion: kubeadm.k8s.io/v1beta3
    kind: ClusterConfiguration
    controllerManager:
      extraArgs:
        cluster-signing-cert-file: /etc/kubernetes/pki/ca.crt
        cluster-signing-key-file: /etc/kubernetes/pki/ca.key
    ```

- 创建证书签名请求（CSR，certificate signing requests）

  详情参考[证书签名请求](./Certificate Signing Requests.md)



# 用外部CA更新证书























