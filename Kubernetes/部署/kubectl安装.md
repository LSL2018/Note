# kubectl安装

## 1）下载最新的版本：

```shell
curl -L0 "[https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl](https://dl.k8s.io/release/$(curl -L -s https:/dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl)"
```

想下载指定版本，用指定版本号。例如下载v1.22.0：

```shell
curl -L0 https://dl.k8s.io/release/v1.22.0/bin/linux/amd64/kubectl
```

 

## 2）验证二进制文件（可选）:

- 下载kubectl校验和文件：

  ```shell
  curl -L0 "[https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl](https://dl.k8s.io/release/$(curl -L -s https:/dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl)"
  ```

- 验证：

  ```shell
  echo "$(<kubectl.sha256) kubectl" | sha256sum --check
  ```

  如果没问题，输出为：

  ```
  kubectl: OK
  ```

## 3）安装kubectl：

```shell
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

## 4）查看版本：

```shell
kubectl version --client
```

#