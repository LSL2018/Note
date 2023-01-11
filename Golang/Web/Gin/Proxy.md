# 设置信任的网段或网址

- 默认情况下，如果没有指定受信任的proxy，Gin信任所有proxy，这是不安全的。

- 如果不使用任何proxy，可以使用engine . SetTrustedProxies(nil)禁用该特性，然后Context.ClientIP()将直接返回远程地址，以避免一些不必要的计算。

- 示例：

  ```go
  import (
    "fmt"
  
    "github.com/gin-gonic/gin"
  )
  
  func main() {
  
    router := gin.Default()
    router.SetTrustedProxies([]string{"192.168.1.2"})
  
    router.GET("/", func(c *gin.Context) {
      //；
      fmt.Printf("ClientIP: %s\n", c.ClientIP())
    })
    router.Run()
  }
  ```

  - 在router.ForwardedByClientIP=true时，SetTrustedProxies设置一个受信任的ip列表
  - ClientIP()用于返回真实的客户端ip，底层调用RemoteIP()，用于检查远端ip是否是受信任的：
    - 如果是，会尝试解析Engine.RemoteIPHeaders中定义的header（默认是[X-Forwarded-For, X-Real-Ip]），从而推断出原始的客户端ip
    - 否则，返回Request.RemoteAddr的ip