

- 下载Swag

  ```go
  #Go 1.17前：
    go get -u github.com/swaggo/swag/cmd/swag
  
  #从Go 1.17开始，用下述命令下载：
    go install github.com/swaggo/swag/cmd/swag@latest
  ```

- 初始化

  ```shell
  #swag.exe在$GOPATH/bin下，
  #需要在项目的main.go所在路径下执行该命令：
    $GOPATH\bin\swag init
  ```

  疑问：为什么有些可以直接执行swag init？是环境变量的配置问题吗？

  

- 下载gin-swagger

  ```shell
  go get -u github.com/swaggo/gin-swagger
  go get -u github.com/swaggo/files
  ```

  
