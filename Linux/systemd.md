# 由来

- 历史上，linux的启动一直采用init进程，比如：

  ```shell
  sudo /etc/init.d/apache2 start
  或者
  service apache2 start
  ```

  这样的启动方法有两个缺点：

  - 启动时间长，因为init进程是串行启动
  - 启动脚本复杂，因为init进程只是执行启动脚本，不管其他事情，脚本需要自己处理各种情况



# 基本概念

- 守护进程(daemon)是一直在后台运行的特殊进程，用于执行特定的系统任务：很多守护进程在系统引导的时候启动，并且一直运行直到系统关闭；另一些只在需要的时候才启动，完成任务后就自动结束
- 字母d是守护进程（daemon）的缩写，Systemd 这个名字的含义就是守护整个系统
- 取代了initd，成为系统的第一个进程（PID=1），其余进程都是它的子进程
- 架构图：

![img](C:\Users\liaosl\OneDrive\笔记\Linux\bg2016030703.png)