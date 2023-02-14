- Kubernetes要求使用PKI证书进行TLS身份验证
- 如果使用kubeadm安装Kubernetes，会自动生成集群所需的证书
- 可以生成自己的证书，例如，不将私有密匙存储在API服务器上而使它们更加安全

# 集群如何使用证书

Kubernetes的以下操作需要PKI支持：

- kubelet要向API服务器进行身份验证的客户端证书
- API服务器端点的服务器证书
- 用于集群管理员向API服务器进行身份验证的客户端证书
- 用于API服务器与kubelet通信的客户端证书
- 用于API服务器与etcd对话的客户端证书
- 用于控制器管理器与API服务器对话的客户端证书/ kubecconfig
- 用于调度程序与API服务器对话的客户端证书/ kubecconfig
- 前端代理的客户端和服务器证书

# 证书存储位置

- 用Kubeadm安装Kubernetes集群，大部分证书都存储在/etc/kubernetes/pki
- kubeadm把用户账户证书放在/etc/kubernetes

# 手动配置证书

​	如果不希望kubeadm生成所需的证书，可以使用single root CA（certificate authority）或提供所有证书来创建它们。

## Single root CA

​	可以创建一个single root CA，由管理员控制。这个CA可以创建多个中间CA，并将所有进一步的拆功能键委托给Kubernetes本身。

​	需要这些CA：

<img src="C:\Users\liaosl\OneDrive\笔记\Kubernetes\网络\image-20211027140253447.png" alt="image-20211027140253447" style="zoom:80%;" />

​	除了这些CA以外，还需要获取用于服务账户管理的密钥对，即sa,key和sa.pub。

​	示例如下：

<img src="C:\Users\liaosl\OneDrive\笔记\Kubernetes\网络\image-20211027140447703.png" alt="image-20211027140447703" style="zoom:80%;" />

## 所有的证书

​	如果不想将CA私钥复制到集群中，可以自己生成所有的证书。



## 证书路径



# 配置用户账号证书

以下管理员账户和服务账户需要手动配置：

<img src="C:\Users\liaosl\OneDrive\笔记\Kubernetes\证书管理\image-20211028144352186.png" alt="image-20211028144352186" style="zoom:67%;" />



- kubelet.conf 中 <nodeName> 的值 **必须** 与 kubelet 向 apiserver 注册时提供的节点名称的值完全匹配

- 对于每个配置，使用给定的CN和O生成X509证书/密钥对

- 为每个配置运行：

  ```shell
  KUBECONFIG=<filename> kubectl config set-cluster default-cluster --server=https://<host ip>:6443 --certificate-authority <path-to-kubernetes-ca> --embed-certs
  KUBECONFIG=<filename> kubectl config set-credentials <credential-name> --client-key <path-to-key>.pem --client-certificate <path-to-cert>.pem --embed-certs
  KUBECONFIG=<filename> kubectl config set-context default-system --cluster default-cluster --user <credential-name>
  KUBECONFIG=<filename> kubectl config use-context default-system
  ```

  上述文件用途：

  <img src="C:\Users\liaosl\OneDrive\笔记\Kubernetes\证书管理\image-20211028152712473.png" alt="image-20211028152712473" style="zoom:67%;" />

  文件完整路径：

  ```shell
  /etc/kubernetes/admin.conf
  /etc/kubernetes/kubelet.conf
  /etc/kubernetes/controller-manager.conf
  /etc/kubernetes/scheduler.conf
  ```

  
