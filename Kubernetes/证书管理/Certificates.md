# 综述	

​	使用客户端证书身份验证时，可以通过easyrsa，openssl或cfssl手动生成证书。

​	

# easyrsa

- 下载，解压并初始化已打过补丁的easyrsa3版本

  ```shell
  curl -LO https://storage.googleapis.com/kubernetes-release/easy-rsa/easy-rsa.tar.gz
  tar xzf easy-rsa.tar.gz
  cd easy-rsa-master/easyrsa3
  ./easyrsa init-pki
  ```

- 生成新的证书颁发机构（CA）

  ```shell
  # --batch设置自定模式
  # --req-cn指定CA的新root证书的CN（Common Name）
  
  ./easyrsa --batch "--req-cn=${MASTER_IP}@`date +%s`" build-ca nopass
  ```

- 生成服务器证书和密钥

  ```shell
  # --subject-alt-name设置API服务器访问时可能使用的ip和DNS名字
  # MASTER_CLUSTER_IP通常时服务CIDP的第一个I
  
  ./easyrsa --subject-alt-name="IP:${MASTER_IP},"\
  "IP:${MASTER_CLUSTER_IP},"\
  "DNS:kubernetes,"\
  "DNS:kubernetes.default,"\
  "DNS:kubernetes.default.svc,"\
  "DNS:kubernetes.default.svc.cluster,"\
  "DNS:kubernetes.default.svc.cluster.local" \
  --days=10000 \
  build-server-full server nopass
  ```

  



# openssl





# cfssl





# 分发自签名CA证书