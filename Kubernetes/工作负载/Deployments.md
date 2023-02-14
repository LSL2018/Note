

# 概念

1）Deployment为Pods和ReplicaSets提供声明式更新（declarative update）

2）在Deployment中描述所需的状态，Deployment控制器会将实际状态更改为所需状态；

3）通过定义Deployment可以创建新的ReplicaSet，删除现有的Deployment，并使用新的Deployment接受它们的资源；

4）不要管理Deployment拥有的ReplcaSet



# 创建Deployment

- 生成nginx-deployment.yaml

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment
    labels:
      app: nginx
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
          - containerPort: 80
  ```

- 创建Deployment（确保k8s集群已启动并运行）

  ```shell
  kubectl apply -f nginx-deployment.yaml
  ```

- 验证Deployment是否创建成功

  ```shell
  kubectl get deployment
  ```

- 查看Deployment的rollout status

  ```shell
  kubectl rollout status
  ```

- 查看Deployment创建的ReplicaSet(rs)

  ```shell
  kubectl get rs
  ```

- 查看为每个Pod自动生成的标签

  ```shell
  kubectl get pods --show-labels
  ```

​    在Deployment中必须指定一个合适的Selector和Pod模板标签(在本例中是app: nginx)；不要将标签或选择器与其他控制器(包括其他Deployment和StatefulSets)重叠。Kubernetes不能阻止重叠，如果多个控制器有重叠的选择器，这些控制器可能会冲突并表现出意外。

# 更新Deployment

​	当且仅当Deployment的Pod模板(即.spec.template)被更改时，例如，如果模板的标签或容器图像被更新，才会触发Deployment的rollout。其他更新(如扩展Deployment)不会触发推出。

​	

​	更新步骤如下（以更新nginx Pods为例）

- 更新nginx Pod，使用的nginx image从1.14.2到1.16.1

  ```shell
  kubectl deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
  或
  kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
  ```

  或者修改nginx-deployment.yaml里的.spec.template.spec.containers[0].image，从1.14.2到1.16.1

  ```shell
  kubectl edit deployment.v1.apps/nginx-deployment
  ```

- 查看rollout status

  ```shell
  kubectl rollout status deployment/nginx-deployment
  ```

- 查看Deployment的详细信息

  ```shell
  kubectl describe deployment
  ```

# 回滚Deployment

​	默认情况下，所有Deployment的rollout历史都保存在系统中，以便可以随时回滚。

​	当触发Deployment的rollout时，将创建Deployment的修订（revision）,这意味着当且仅当Deployment的Pod模板(.spec.template)被更改时，才会创建新的修订版本，这意味着当回滚到以前的版本时，只回滚Deployment的Pod模板部分。

## 查看Deployment的rollout历史记录

- 检查Deployment的修订：

  ```shell
  kubectl rollout history <Deployment名>
  ```

- 查看某个修订的具体信息：

  ```shell
  kubectl rollout history <Deployment名> --revision=<revision id>
  ```

## 回滚到上一个版本

- 撤销当前的rollout，回滚到上一个版本

  ```shell
  kubectl rollout undo <Deployment>
  ```

  或者回滚到指定版本：

  ```shell
  kubectl rollout undo <Deployment> --to-revision=<revision id>
  ```

# 扩展Deployment

- 增加副本数（比如：10）

  ```shell
  kubectl scale <Deployment> --replicas=10
  ```

- 设置自动伸缩器，根据现有Pod的CPU利用率(比如：80%)来选择希望运行的Pod的最小（比如：10）和最大值（比如：15）

  ```shell
  kubectl autoscale <Deployment> --min=10 --max=15 --cpu-percent=80
  ```

  注意：需要先在集群启用水平Pod自动伸缩（horizontal Pod autoscaling）

滚动更新的时候可以同时运行一个应用程序的多个版本



# 暂停和恢复Deployment

​	可以在触发一个或多个更新之前暂停Deployment，然后恢复Deployment。这样可以在暂停和恢复之间应用多个修复，不会触发不必要的rollout。

- 暂停Deployment

  ```shell
  kubectl rollout pause <Deployment>
  ```

- 恢复Deployment

  ```shell
  kubectl rollout resume <Deployment>
  ```

在恢复已暂停的Deployment之前，无法回滚

# Deployment status

​	可以通过以下命令来查看Deployment的进度：

```shell
kubectl rollout status <Deployment>
```

- Progressing

  - 创建新的ReplicaSet

  - 扩展最新的ReplicaSet

  - 缩小旧的RelicaSet

  - 新的pod已经准备好或可用(至少为MinReadySeconds准备好了)

- Complete

  - 与Deployment关联的所有副本都已更新到指定的最新版本
  - 与Deployment相关联的所有副本都是可用的
  - Deployment没有正在运行的旧副本

- Failed

# 清理

​    通过设置.spec.revisionHistoryLimit来指定该Deployment最多可以保留多少旧的Replicaset,默认10。

​    将该字段设置为0，将导致清除Deployment的所有历史记录，因此Deployment将无法回滚。



# 编写Deployment Spec

- Pod Template
  - 只有.spec.template和.spec.selector是.spec中必需的
  - 是一个Pod模板，和Pod的语法规则完全相同。只是这里它是嵌套的，因此不需要 apiVersion 或 kind
  - 除了Pod所需的字段外，Deployment 中的Pod template还必须指定适当的labels和适当的重启策略。对于label，确保不要与其他控制器重叠
  - .spec.template.spec.restartPolicy默认为Always，不需要指定
- Replicas
  - .spec.replicas是可选字段，指定所需pod的数量，默认值是1
- Selector
  - .spec.selector 必须匹配 .spec.template.metadata.labels，否则请求会被 API 拒绝
- Strategy
  - 指定用新Pod替换旧Pod的策略
  - .spec.strategy.type可以是“Recreate”或“RollingUpdate”(默认)
  - .spec.strategy.type==Recreate：在创建新 Pods 之前，所有现有的 Pods 会被杀死
  - .spec.strategy.type==RollingUpdate：滚动更新，可以指定maxUnavailable和maxSurge
  - .spec.strategy.rollingUpdate.maxUnavailable是可选字段，指定更新过程中不可用的Pod个数上限
  - .spec.strategy.rollingUpdate.maxSurge是可选字段，指定可以创建的超出期望Pod个数的Pod数量
- Progress Deadline Seconds
  - .spec.progressDeadlineSeconds是一个可选字段，用于指定系统在报告 Deployment 之前等待 Deployment 取得进展的秒数
  - 如果指定，则此字段值需要大于 .spec.minReadySeconds
- Min Ready Seconds
  - .spec.minReadySeconds是一个可选字段，用于指定新创建的 Pod 在没有任意容器崩溃情况下的最小就绪时间
  - 只有超出这个时间 Pod 才被视为可用。默认值为 0（Pod 在准备就绪后立即将被视为可用）
- Revision History Limit
  - 参考[清理](#清理)
- Paused
  - .spec.paused 是用于暂停和恢复 Deployment 的可选字段
  - 暂停的 Deployment 和未暂停的 Deployment 的唯一区别是：Deployment 处于暂停状态时， PodTemplateSpec 的任何修改都不会触发新的rollout；Deployment 在创建时是默认不会处于暂停状态

