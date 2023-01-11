# minikube搭建

## 前提

​		docker已安装

​		kubectl已安装

​		防火墙关闭（systemctl status firewalld：查看防火墙状态）

## 安装（官网，root用户）：

```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
install minikube-linux-amd64 /usr/local/bin/minikube
```

##  启动（非root用户）：

```shell
minikube start
```

## 卸载（root用户）：

```shell
## 第一步：停止运行
	minikube stop
	
## 第二步：卸载
	minikube delete
	docker stop (docker ps -aq)
	rm -r ~/.kube ~/.minikube
	rm /usr/local/bin/localkube /usr/local/bin/minikube
	systemctl stop 'kubelet.mount'
	rm -rf /etc/kubernetes/
	docker system prune -af --volumes
```

