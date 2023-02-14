# Cgroup驱动配置

## 前提

​	熟悉Runtime

## 配置Kubelet的Cgroup驱动

1）kubeadm允许在kubeadm初始化过程中传递一个kubeletconfiguration结构。KubeletConfiguration可以包含cgroupDriver字段，该字段控制kubelet的cgroup驱动程序。在v1.22中，如果用户没有在KubeletConfiguration下设置cgroupDriver字段，kubeadm将默认它为systemd。

2）Kubeadm对集群中的所有节点使用相同的KubeletConfiguration。KubeletConfiguration存储在kube-system命名空间下的ConfigMap对象中。

3）执行init、join和upgrade子命令会导致kubeadm将KubeletConfiguration作为文件写入/var/lib/kubelet/config。并将其传递给本地节点kubelet。

4）配置示例：

```yaml
# kubeadm-config.yaml
kind: ClusterConfiguration
apiVersion: kubeadm.k8s.io/v1beta3
kubernetesVersion: v1.21.0
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
```

5）传递配置参数：

```yaml
kubeadm init --config kubeadm-config.yaml
```





## 迁移到systemd驱动

​	将现有的kubeadm集群的cgroup驱动更改为systemd，需要一个类似于kubelet升级的过程：

- 修改kubelet的ConfigMap:

  - 查找kubelet的ConfigMap名称：

    ```shell
    kubectl get cm -n kube-system | grep kubelet-config
    ```

  - 执行以下命令：

    ```shell
    kubectl edit cm kubelet-config-<kubernetes version> -n kube-system
    ```

  - 修改或新增以下字段：

    ```yaml
    cgroupDriver: systemd
    ```

    - 该字段必须出现在ConfigMap的"kubelet:"部分下

- 更新所有节点上的cgroup驱动（对集群中的每个节点）：

  - 清空节点：

    ```shell
    kubectl drain <node-name> --ignore-daemonsets
    ```

  - 停止kubelet

    ```shell
    systemctl stop kubelet
    ```

  - 停止container runtime

  - 修改container runtime的cgroup驱动为systemd

  - 设置/var/lib/kubelet/config.yaml：

    ```yaml
    cgroupDriver: systemd
    ```

  - 启动container runtime

  - 启动kubelet

    ```shell
    systemctl start kubelet
    ```

  - uncordon节点：

    ```shell
    kubectl uncordon <node-name>
    ```

  

  

  

  

  
