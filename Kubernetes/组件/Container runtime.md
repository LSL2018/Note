# 综述

- Control groups（控制组）用于约束分配给进程的资源

- 当systemd（[Systemd相关](onenote:https://d.docs.live.net/11df3f0f9b007b1f/文档/Linux/命令行.one#Systemd相关&section-id={8500CEEC-7692-4927-BE06-1ADF5EAAA058}&page-id={3AB25B10-28F0-456F-8A0C-FD89A42E2B75}&end)）被选为Linux发行版的init系统时，init进程生成并消耗一个root cgroup，并充当cgroup管理器。

- Systemd与cgroups紧密集成，并为每个Systemd单元分配一个cgroup。

- 可以配置container runtime和kubelet来使用cgroupfs。

- systemd和cgroupfs一起使用意味着将有两个不同的cgroup管理器。

- 单个cgroup管理器简化了正在分配资源的视图，并且默认情况下对可用资源和正在使用的资源有更一致的视图。当一个系统上有两个cgroup管理器时，将得到这些资源的两个视图。在这个领域，有人报告过这样的情况:对kubelet和Docker使用cgroupfs，而对其余进程使用systemd的节点，在资源压力下变得不稳定。

- 修改已加入集群的节点的cgroup驱动是一项敏感操作。如果kubelet使用一个cgroup的语义创建了Pod，那么在试图为现有的Pod重新创建Pod沙箱时，将container runtimes更改为另一个cgroup可能会导致错误。重新启动kubelet可能无法解决这些错误。

- 如果有了自动化，可以使用更新后的配置将该节点替换为其他节点，或者使用自动化重新安装该节点。

- 即使内核支持混合配置，其中一些控制器由cgroup v1管理，另一些由cgroup v2管理，Kubernetes也只支持同一个cgroup版本来管理所有的控制器。

- 如果systemd默认不使用cgroup v2，可以添加以下命令来使用：

  ```shell
  # dnf install -y grubby && \
  sudo grubby \
  --update-kernel=ALL \
  --args="systemd.unified_cgroup_hierarchy=1"
  ```



 

# **Containerd**

## 1）安装及配置先决条件

```shell
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

*# Setup required sysctl params, these persist across reboots.*
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

*# Apply sysctl params without reboot*
sudo sysctl --system
```

## 2）安装containerd



- 安装containerd.io：

  ```shell
  sudo yum -y install container.io
  ```

- 配置containerd.io：

  ```shell
  sudo mkdir -p /etc/containerd
  containerd config default | sudo tee /etc/containerd/config.toml
  ```

- 重启containerd：

  ```shell
  sudo systemctl restart containerd
  ```

  

## 3）使用systemd作为cgroup驱动程序

- 在/etc/containerd/config.toml，设置：

  ```shell
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
   ...
   [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
  ```



- 重启:

  ```shell
  sudo systemctl restart containerd 
  ```

  

# **CRI-O**

## 1）安装及配置先决条件

```shell
- # Create the .conf file to load the modules at bootup
cat <<EOF | sudo tee /etc/modules-load.d/crio.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Set up required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward         = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```



## 2）环境变量配置：

- os=<操作系统>，例如：Centos_7
- VERSION=<cri-o版本>，例如：1.20:1.20.0

## 3）安装cri-o：

```shell
sudo curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable.repo https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/devel:kubic:libcontainers:stable.repo

sudo curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo

sudo yum -y install cri-o
```

 

## 4）启动并设置开机自启：

```shell
sudo systemctl daemon-reload
sudo systemctl enable crio --now
```



## 5）使用systemd作为cgroup驱动程序

- cri-o默认使用systemd cgroup驱动程序。

- 要切换到cgroupfs的cgroup驱动，可以编辑/etc/crio/crio.conf或在/etc/crio/crio.conf.d/02-cgroup-manager.conf中插入配置，例如:

  ```sh
  [crio.runtime]
  conmon_cgroup="pod"
  cgroup_manager="cgroupfs"
  ```



# Docker

## 1）在每个节点上安装Docker



## 2）配置Docker daemon

```shell
sudo mkdir /etc/docker

cat <<EOF | sudo tee /etc/docker/daemon.json
{
 "exec-opts": ["native.cgroupdriver=systemd"],
 "log-driver": "json-file",
 "log-opts": {
  "max-size": "100m"
 },
 "storage-driver": "overlay2"
}
EOF

```



## 3）重启并设置开机自启动：

```shell
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```

 

 