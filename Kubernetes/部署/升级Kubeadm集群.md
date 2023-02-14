# 综述

- 高层次升级流程
  - 升级控制平面主节点
  - 升级其他控制平面节点
  - 升级工作节点
- 集群使用静态控制平面和etcd pods，或者外部etcd
- 升级前备份重要组件
- 关闭Swap
- 升级后重启所有容器，因为容器规约的哈希值被改变了

# 升级步骤

## 确定升级版本

- Ubuntu,Debian,或者HypriotOS:

  ```shell
  apt update
  apt-cache madison kubeadm
  ```

- CentOS,RHEL,或者Fedora

  ```shell
  yum list --showdiplicates kubeadm --disableexcludes=kubernetes
  ```

  上述命令用于寻找最新的1.22版本，结果应该类似于1.22.x-0



## 升级控制平面节点

​	控制平面节点的升级是逐个执行，需要/etc/kubernetes/admin.conf。

​	步骤：

- 对第一个控制平面节点：

  - 升级Kubeadm:

    - Ubuntu,Debian,或者HypriotOS:

      ```shell
      # 用最新的发行版本号替换1.22.x-00中的x
      apt-mark unhold kubeadm && \
      apt-get update && apt-get install -y kubeadm=1.22.x-00 && \
      apt-mark hold kubeadm
      -
      # since apt-get version 1.1 you can also use the following method
      apt-get update && \
      apt-get install -y --allow-change-held-packages kubeadm=1.22.x-00
      ```

    - CentOS,RHEL,或者Fedora：

      ```shell
      yum install -y kubeadm-1.22.x-0 --disableexcludes=kubernetes
      ```

  - 验证：

    ```shell
    kubeadm version
    ```

  - 检查集群是否可以升级，并获取可以升级到的版本：

    ```shell
    kubeadm upgrade plan
    ```

- 

