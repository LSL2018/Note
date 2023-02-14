

#  理论

- goroutine是Go并行设计的核心。

- goroutine说到底其实就是协程（比线程更小）

- 执行goroutine只需极少的栈内存(大概是4~5KB)，会根据相应的数据伸缩

- 通过 go 关键字启动：

  ```go
  go hello(a, b, c)
  ```

- 遵循：不要通过共享来通信，而要通过通信来共享

- main函数也是用goroutine调用的，当main函数返回后，所有goroutine都暴力终结



# 控制并发数

