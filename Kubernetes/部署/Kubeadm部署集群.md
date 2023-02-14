# 0 安装要求

部署Kubernetes集群机器需要满足以下几个条件：

- 一台或多台机器,操作系统 CentOS7.x-86_x64

- 硬件配置：2GB或更多RAM,2个CPU或更多CPU,硬盘30GB或更多

- 可以访问外网,需要拉取镜像,如果服务器不能上网,需要提前下载镜像并导入节点

禁止swap分区

 

# 1 环境准备（所有节点）

## 1）基础配置

```shell
#配置hosts
cat >> /etc/hosts << EOF
192.168.27.132 k8s0
192.168.27.133 k8s1
192.168.27.134 k8s2
EOF

#关闭防火墙
systemctl stop firewalld && \
systemctl disable firewalld

#关闭selinux
setenforce 0 # 临时
sed -i 's/enforcing/disabled/' /etc/selinux/config # 永久

#关闭swap
swapoff -a # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab  # 永久

#安装ssh服务
yum install -y openssl openssh-server
yum install rsync -y
用vim打开配置文件/etc/ssh/sshd_config，找到PermitRootLogin将其前面的注释去掉
systemctl start sshd.service && systemctl enable sshd.service
```

## 2）将桥接的IPv4流量传递到iptables的链

```shell
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

## 3）时间同步

```shell
#安装ntpdate
yum install ntpdate -y

#不同机器之间的同步
#ntpdate [-nv] [NTP IP/hostname]
ntpdate time.nist.gov
```



# 2 安装

## 1）安装keepalived（可选）

## 2）部署haproxy（可选）

## 3）安装Docker（所有节点）

```shell
#卸载旧版本
    sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine

#安装Docker Engine-Community
## 第一步：设置仓库
    sudo yum install -y yum-utils device-mapper-persistent-data lvm2
	sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo	
## 第二步：安装
	sudo yum -y install docker-ce docker-ce-cli containerd.io

#启动服务
systemctl start docker && systemctl enable docker
#查看服务
systemctl status docker
#查看版本
docker --version

#registry-mirrors：配置Docker镜像源
#exec-opts：配置docker的cgroup，和kubelet默认的一致
mkdir /etc/docker
cat > /etc/docker/daemon.json << EOF 
{ 
 "registry-mirrors":["https://qt8avii4.mirror.aliyuncs.com"],
 "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

#加载镜像喝重启服务
systemctl daemon-reload && systemctl restart docker
```

## 4）安装kubeadm,kubelet和kubectl（所有节点）

- kubernetes.repo配置

  ```shell
  cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
  [kubernetes]
  name=Kubernetes
  baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
  enabled=1
  gpgcheck=1
  repo_gpgcheck=1
  gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg 
  exclude=kubelet kubeadm kubectl
  EOF
  ```

- 安装

  ```shell
  sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
  ```

- kubeadm初始化

  ```shell
  # kubeadm-config.yaml
  kind: ClusterConfiguration
  apiVersion: kubeadm.k8s.io/v1beta3
  kubernetesVersion: v1.23.4
  imageRepository: registry.aliyuncs.com/google_containers
  networking:
    podSubnet: 192.168.0.0/16
    
  #执行
  kubeadm init --config=kubeadm-config.yaml
  ```
  
  注：在启动kubelet前需要kubeadm init，不然会出现“error: open /var/lib/kubelet/config.yaml: no such file or directory”的错误
  
  
  
- 设置kubelet自启动

  ```shell
  sudo systemctl enable --now kubelet
  ```

- 让kubectl为非root用户工作:

  ```shell
  mkdir -p $HOME/.kube
  
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```

- 让kubectl为root用户工作（root用户每次启动k8s前都要执行）：	

  ```shell
  export KUBECONFIG=/etc/kubernetes/admin.conf
  ```


## 5）将节点加入集群

### 1）主节点（同时作为控制节点）

执行kubeadm init输出的命令，例如：

```shell
kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash> --control-plane

## token可以通过以下命令获取：
	kubeadm token list
## token创建（24小时有效期）：
	kubeadm token create
## --discovery-token-ca-cert-hash通过以下命令获取：
	openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
    openssl dgst -sha256 -hex | sed 's/^.* //'
```

查看集群状态：

```shell
kubectl get cs

kubectl get pods -n kube-system

kubectl get node
```

安装flannel网络,用于节点之间的通信：

```shell
mkdir /usr/local/kubernetes/manifests/flannel
cd /usr/local/kubernetes/manifests/flannel

#获取kube-flannel.yml 
wget -c https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

#执行安装命令
kubectl apply -f kube-flannel.yml 

#查看
kubectl get pods -n kube-system
```

### 2）工作节点（例如：worker01）

拷贝证书：

```shell
#在worker01中：
mkdir -p /etc/kubernetes/pki/etcd
#在master节点中：
  scp /etc/kubernetes/admin.conf root@<worker01 ip>:/etc/kubernetes
  scp /etc/kubernetes/pki/{ca.*,sa.*,front-proxy-ca.*} root@<worker01 ip>:/etc/kubernetes/pki
  scp /etc/kubernetes/pki/etcd/ca.* root@<worker01 ip>:/etc/kubernetes/pki/etcd
```

加入集群：

```shell
#token和hash在主节点处可获取
kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

修改roles（可选？）：

```shell
#最后面的=号表示在原来ROLES基础上再增加一个，-号就表示删除某个ROLES
#node name通过kubectl get nodes查看
#role示例：worker
kubectl label nodes <node name> node-role.kubernetes.io/<role>=
```



# 集群网络重新安装

```shell
kubectl delete -f kube-flannel.yml
kubectl apply -f kube-flannel.yml
```

# 检查状态

```shell
kubectl get node

kubectl get pods --all-namespaces
```



# 测试



