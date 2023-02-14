# Gin概念

- 用golang编写的web框架

- 是一个类似martini的API，但因为httprouter的原因，性能提高了40倍

- 默认启用MsgPack渲染功能

  - MsgPack是一种二进制序列化格式，类似JSON，但更快、更小

  - 可以通过以下操作禁止：

    ```shell
    go build -tags=nomsgpack .
    ```

    