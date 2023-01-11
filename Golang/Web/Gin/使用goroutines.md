在使用goroutines时，不能使用原始的gin.Context，应该使用read-only副本：

```go
func main() {
  r := gin.Default()

  r.GET("/long_async", func(c *gin.Context) {
    // 创建只读副本
    cCp := c.Copy()
    go func() {
      time.Sleep(5 * time.Second)
      //使用的是副本
      log.Println("Done! in path " + cCp.Request.URL.Path)
    }()
  })

  r.GET("/long_sync", func(c *gin.Context) {
    // simulate a long task with time.Sleep(). 5 seconds
    time.Sleep(5 * time.Second)

    // 没有使用goroutine，所以不需要使用副本
    log.Println("Done! in path " + c.Request.URL.Path)
  })

  // Listen and serve on 0.0.0.0:8080
  r.Run(":8080")
}
```

