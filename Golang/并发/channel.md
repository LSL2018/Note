# 理论

- goroutine之间可以通过channel发送或者接收值。这些值只能是特定的channel类型
- 复制或作为参数传递时，复制的是引用
- 零值是nil
- 同种类型的channel可以用==进行比较
- 默认情况下，channel接收和发送数据都是阻塞的，除非另一端已经准备好，因此goroutines的同步不需要显式的lock

# 类型

## 非缓存型

- 发送操作将会阻塞，直到另一个goroutine执行接收操作

- 如果接收操作先执行，接收方goroutine将阻塞，直到另一个goroutine在同一个通道执行发送操作

- 使用：

  ```go
  //创建
  ci := make(chan int)
  cs := make(chan string)
  cf := make(chan interface{})
  
  //发送和接收
  ch <- v    // 发送v到channel ch.
  v := <-ch  // 从ch中接收数据，并赋值给v
  close(ch)  //关闭channel
  
  
  //参数化：out只允许发送，in只允许接收
  func xxx(out chan<- int, in <-chan int) {
      ……
  }
  ```
  
  

## 缓存型

- 指定channel的缓冲大小，即channel可以存储的数据量

- 

- 使用：

  ```go
  //创建可以存储4个元素的int型channel
  //在这个channel 中，前4个元素可以无阻塞的写入。当写入第5个元素时，代码将会阻塞，直到其他goroutine从channel 中读取一些元素
  ch := make(chan int, 4)
  cap:=cps(ch) //获取通道总容量
  len:=len(ch) //获取当前通道元素个数
  ```
  
  