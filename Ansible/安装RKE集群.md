# 0 准备工作

注意：centos不能用root用户执行docker命令，因此需要创建其他用户

## 0 节点信息

| IP             | hostname        |
| -------------- | --------------- |
| 192.168.27.140 | ansible-manager |
| 192.168.27.135 | rancher-node01  |
| 192.168.27.142 | rancher-node02  |
| 192.168.27.141 | rancher-node03  |

- hostname可以通过以下命令修改：

  ```shell
  hostnamectl set-hostname <hostname>
  ```

## 1 配置hosts文件（ansible-manager）

```shell
cat >> /etc/hosts << EOF
192.168.27.140 ansible-manager
192.168.27.135 rancher-node01
192.168.27.142 rancher-node02
192.168.27.141 rancher-node03
EOF
```

## 2 配置主机清单（ansible-manager）

```shell
#在/etc/ansible/hosts底部添加：
[rancher]
192.168.27.135
192.168.27.142
192.168.27.141
```

- /etc/ansible/hosts 是主机清单配置文件，由 ansible.cfg文件中的 inventory 变量配置，默认值为/etc/ansible/hosts
- 在使用ansible命令前，需要对hosts文件进行相关主机清单配置
- 使用说明：
  - 可以不对主机进行分组，如果不指定分组，需要配置在所有的分组前
  - 可以对主机进行分组，中括号里包含的名字代表组名
  - 主机可以使用域名，主机名，ip地址表示，一般以IP居多

## 3 设置免密登录（ansible-manager）

```shell
ssh-keygen -t rsa

ssh-copy-id rancher-node01
ssh-copy-id rancher-node02
ssh-copy-id rancher-node03
```

## 4 测试（ansible-manager）

执行以下命令

```shell
ansible all -m ping
```



# 1 基础配置

配置hosts文件，关闭防火墙、selinux和swap

ntpdate时间同步（定时）

```shell
#vim init.yml
---
- hosts: rancher
  vars:
  remote_user: rancherusr
  gather_facts: false
  
  tasks:
  - name: update hosts
    lineinfile: dest=/etc/hosts line="{{item.ip}} {{item.hostname}}"
    with_items:
      - {ip: '192.168.27.135', hostname: 'rancher-node01'}
      - {ip: '192.168.27.142', hostname: 'rancher-node02'}
      - {ip: '192.168.27.141', hostname: 'rancher-node03'}
 
  tasks:
  - name: stop firewall
    service: name=firewalld state=stopped enabled=no
 
  - name: stop selinux
    lineinfile:
      dest: /etc/selinux/config
      regexp: "^SELINUX="
      line: "SELINUX=disabled"
  
  - name: 即时生效
    shell: setenforce 0 ; swapoff -a
 
  - name: 关闭swap
    lineinfile:
      dest: /etc/fstab
      regexp: ".*swap"
      line: ""
         

```



# 2 ntpdate时间同步

```yaml
#vim ntpdate.yml
---  
- hosts: rancher
  any_errors_fatal: "{{ any_errors_fatal | default(true) }}"
  tasks:
    - name: yum install ntp
      yum:
        name: ntp
        state: present
    
    - name: set timezone to Asia-Shanghai
      shell: /usr/bin/timedatectl set-timezone Asia/Shanghai
        
    - name: stop ntpd service
      systemd:
        name: ntpd
        daemon_reload: yes
        state: stopped
      tags:
        - stop__ntpd

    - name: manual sync time with ntpdate
      shell: /usr/sbin/ntpdate ntp1.aliyun.com
      tags:
        - manual_sync_datetime

    - name: enable and start ntpd
      systemd:
        name: ntpd
        daemon_reload: yes
        state: started
        enabled: yes
      tags:
        - enable_start_ntpd

```



# 3 docker安装

```json
#vim daemon.json
#和docker_install.yaml同级目录
{
    "registry-mirrors": ["https://qt8avii4.mirror.aliyuncs.com"],
    "exec-opts":["native.cgroupdriver=systemd"]
}
```

```yaml
#vim docker_install.yaml
---
- hosts: rancher
  remote_user: root
  tasks:
    - name: uninstall older version
      yum:
        name: 
          - docker,docker-client,docker-client-latest,docker-common,docker-latest,docker-latest-logrotate,docker-logrotate docker-engine
        state: removed
  
    - name: install yum-utils, device-mapper-persistent-data, lvm2
      yum: 
        name: 
          - yum-utils,device-mapper-persistent-data,lvm2
        state: present
      
    - name: add docker repo
      shell: yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
      
    - name: install docker-ce, docker-ce-cli, containerd.io
      yum:
        name: 
          - docker-ce,docker-ce-cli,containerd.io
        state: present
    
    - name: mkdir /etc/docker
      shell: mkdir -p /etc/docker
    
    - name: config mirro
      copy: src=./daemon.json dest=/etc/docker/daemon.json
      
    - name: start enable docker
      service: name=docker state=started enabled=true
      
    - name: restrat
      shell: systemctl daemon-reload && systemctl restart docker
      tags: restart
```



# 4 安装kubectl

```yaml
#vim kubectl_install.yml
---
- hosts: rancher
  remote_user: root
  tasks:
    - name: download latest kubectl document
      shell: curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    
     - name: install kubectl
       shell: install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

# 5 安装RKE

```yaml
#vim rke_install.yml
#先把rke_linux_amd64下载到rancher节点的想要的位置，例如：
#    /app/rancher
---
- hosts: rancher
  remote_user: root
  tasks:
    - name: transform installation package to executable file
      shell: cd /app/rancher && mv rke_linux-amd64 rke && chmod +x rke
    
     

```

查看rke版本：

```shell
#在rke执行文件的路径下：
	./rke --version
```

# 6 生成配置文件

```shell
#vim rancher-cluster.yml
#先把该文件放到rancher节点的想要的位置，例如：
#    /app/rancher
nodes:
  - address: 192.168.27.135
    user: root
    role: [controlplane, worker, etcd]
  - address: 192.168.27.142
    user: root
    role: [controlplane, worker, etcd]
  - address: 192.168.27.141
    user: root
    role: [controlplane, worker, etcd]

services:
  etcd:
    snapshot: true
    creation: 6h
    retention: 24h

# 当使用外部 TLS 终止，并且使用 ingress-nginx v0.22或以上版本时，必须。
ingress:
  provider: nginx
  options:
    use-forwarded-headers: "true"
```

