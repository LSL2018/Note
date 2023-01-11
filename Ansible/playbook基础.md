# 概念

- Playbook 是一个由一个或多个 play 组成的文件；play 是针对特定主机或主机组执行的一组有序的任务；每个 playbook 必须包含两部分：

  - **hosts：** 运行 playbook 的一组主机
  - **tasks：** 需要在主机上运行的任务

  除了这两个必须选项，还有一些可选项选项，也可能需要包含在 play 中，如：

  - **name：** play 的名称，在运行该 play 时，会在运行过程中显示。
  - **become：** 与配置文件中的 become 作用一样，用于提权，当配置文件中禁用提权时，你想要某个 play 使用提权的话，你可以在 play 中添加 become。

- playbook 以 yaml 格式编写的，通常以 yml 扩展名保存
- yaml 格式使用空格缩进，对于空格的数量没有特别要求，但需要注意：
  - 同一级别内的元素必须使用相同的缩进
  - 对于子项目，缩进必须比父项目多

# 编写 示例

```yaml
it@workstation:~/ansible$ cat test.yml
---
- name: Install Apache
  hosts: servera
  tasks:
    - name: Install apache httpd
      apt:
        name: apache2
        state: present
    - name: Copy using inline content
      copy:
        content: Welcome to
        dest: /var/www/html/index.html
    - name: Start apache service
      service:
        name: apache2
        state: started
        enabled: yes
```

Playbook 以 **---** 开头，用于标记文件开始；

- 第二行的 **name** 为该 play 的名称；

- 第三行的 **hosts** 表示将要运行该 play 的主机；

- 第四行的 **tasks** 表示该 play 将要执行的具体任务；

通过缩进，可以看出 tasks 一共分为三个部分，也就是三个模块，

-  name 是可选选项，建议写上，用来作为说明，并且 会显示在执行过程中；

- name 下面的是模块的名称，在该 play 的 tasks 中一共有三个模块：
  - **apt：** 用于安装软件
  - **copy：** 用于复制文件或内容
  - **service：** 用于操作 service，如启动服务，重启服务等

我们可以通过 **ansible-doc** 来获取更多关于模块的信息

通过ansible-playbook <yaml name>执行

```text
it@workstation:~/ansible$ ansible-playbook test.yml 
BECOME password: 

PLAY [Install Apache] ******************************************************************************

TASK [Gathering Facts] *****************************************************************************
ok: [servera]

TASK [Install apache httpd] ************************************************************************
changed: [servera]

TASK [Copy using inline content] *******************************************************************
changed: [servera]

TASK [Start apache service] ************************************************************************
ok: [servera]

PLAY RECAP *****************************************************************************************
servera                    : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

- 在执行过程中会显示 play 的名称，说明现在执行的是那个 play；

- 每个 play 中会有一个默认的任务，就是获取 facts 信息，在 facts 信息中会保存计算机的系统信息；

- 在执行 tasks 时，会显示当前所执行的 task 的名称，以及执行的状态：如果是 **ok**，则表示系统状态没有任何更改；如果是 **changed**，则表示系统状态发生了改变；

- 最后有一个 执行结果的汇总，有多少个 changed，有多少个 failed，有多个 skipped 等等；

