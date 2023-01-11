# 官方安装版本

```shell
## centos版本
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

# 手动安装

## 1. 卸载旧版本

```shell
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
```

## 2. 安装Docker Engine-Community

```shell
## 第一步：设置仓库
    sudo yum install -y yum-utils device-mapper-persistent-data lvm2
	sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
	
## 第二步：安装
	sudo yum -y install docker-ce docker-ce-cli containerd.io
```

# 卸载Docker

## 1. 删除安装包

```shell
yum remove docker-ce
```

## 2. 删除image、容器、配置文件等内容

```shell
rm -rf /var/lib/docker
```
