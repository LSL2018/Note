# 综述

- kubelet是运行在每个节点上的主要“节点代理”。
- kubelet可以使用以下方式向apiserver注册节点：
  - hostname
  - 覆盖hostname的标志
  - 云提供商的特定逻辑
- kubelet根据PodSpec工作：PodSpec是描述pod的YAML或JSON对象。
- kubelet接受一组通过各种机制(主要是通过apiserver)提供的podspec，并确保这些podspec中描述的容器运行良好。
- kubelet不管理不是由Kubernetes创建的容器。
- 除了从apiserver的PodSpec，有三种方式可以向Kubelet提供容器清单:
  - File：命令行里作为参数传递的路径，此路径下的文件被定期监控以获取更新，周期m默认为20s
  - HTTP endpoint：命令行里作为参数传递的HTTP endpoint，20s检查一次
  - HTTP server：侦听HTTP并响应一个简单的API(目前还未规范)来提交一个新的清单



# 参数

