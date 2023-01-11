



# sync.atomic

## 基础概念

- 将底层硬件提供的原子操作封装成了 Go 的函数
- 直接有底层CPU硬件支持，因而一般要比基于操作系统API的锁方式效率高些
- 1.4 版本中，添加了一个新的类型Value，相当于一个容器，可以被用来“原子地”存储和加载任意类型的值
- atomic.Value类型对外暴露的方法有两个：

  - v.Store(c)：将原始变量c存放到一个atomic.Value类型的v里
  - c=v.Load()：从线程安全的v中读取上一步存放的内容



## atomic.Value

### 底层数据结构

```go
sync/atomic/value.go

type Value struct {
  v interface{}
}

type ifaceWords struct {
  typ  unsafe.Pointer
  data unsafe.Pointer
}
```

### Store()源码解读

```go
sync/atomic/value.go line 45~82

func (v *Value) Store(x interface{}) {
  if x == nil {
    panic("sync/atomic: store of nil value into Value")
  }
  vp := (*ifaceWords)(unsafe.Pointer(v))  // Old value
  xp := (*ifaceWords)(unsafe.Pointer(&x)) // New value
  for {
    typ := LoadPointer(&vp.typ)
    if typ == nil {
      // Attempt to start first store.
      // Disable preemption so that other goroutines can use
      // active spin wait to wait for completion; and so that
      // GC does not see the fake type accidentally.
      runtime_procPin()
      if !CompareAndSwapPointer(&vp.typ, nil, unsafe.Pointer(^uintptr(0))) {
        runtime_procUnpin()
        continue
      }
      // Complete first store.
      StorePointer(&vp.data, xp.data)
      StorePointer(&vp.typ, xp.typ)
      runtime_procUnpin()
      return
    }
    if uintptr(typ) == ^uintptr(0) {
      // First store in progress. Wait.
      // Since we disable preemption around the first store,
      // we can wait with active spinning.
      continue
    }
    // First store completed. Check type and overwrite data.
    if typ != xp.typ {
      panic("sync/atomic: store of inconsistently typed value into Value")
    }
    StorePointer(&vp.data, xp.data)
    return
  }
}

```

### Load()源码解读

```go
sync/atomic/value.go

func (v *Value) Load() (x interface{}) {
  vp := (*ifaceWords)(unsafe.Pointer(v))
  typ := LoadPointer(&vp.typ)
  if typ == nil || uintptr(typ) == ^uintptr(0) {
    // First store not yet completed.
    return nil
  }
  data := LoadPointer(&vp.data)
  xp := (*ifaceWords)(unsafe.Pointer(&x))
  xp.typ = typ
  xp.data = data
  return
}

```







# sync.Map

- golang中map不是线程安全的；sync.map就是1.9版本带的线程安全map

- 内部方法：

  ```go
  //获取key对应的value；如果不存在，则返回nil
  //ok表示key是否存在
  Load(key interface{}) (value interface{}, ok bool)
  
  //新增或更新k-v对
  Store(key, value interface{})
  
  //获取key对应的value；如果不存在，就存储并返回给定的值
  //读取：true；存储：false
  LoadOrStore(key, value interface{}) (actual interface{}, loaded bool)
  
  //删除key
  Delete(key interface{})
  
  //循环读取
  //无法使用for range遍历sync.map
  Range(f func(key, value interface{}) bool)
  ```
  
- 适用于读多写少的场景

- 



## 内部方法

- 







# sync.one







# sync.pool











# sync.Mutex（互斥锁）和sync.RWMutex（读写互斥锁）

- Mutex是最简单粗暴的，当一个 goroutine 获得了 Mutex 后，其他 goroutine 就只能乖乖等到这个 goroutine 释放该 Mutex

- RWMutex是单写多读型：

  - 在读锁占用的情况下，会阻止写，但不阻止读

  - 多个 goroutine 可同时获取读锁（RLock() ）

  - 而写锁（Lock() ）会阻止任何其他 goroutine（无论读和写）进来，整个锁相当于由该 goroutine 独占

  - 底层：

    ```go
    type RWMutex struct {
        w Mutex
        writerSem uint32
        readerSem uint32
        readerCount int32
        readerWait int32
    }
    ```

    

# sync.WaitGroup（等待组）

- 在内部维护一个计数，初始默认值为0；有以下几个方法可用：

  - Add(delta int)：计数+1
  - Done()：计数-1；等价于Add(-1) 
  - Wait()：计数不为0时阻塞，直到为0
    - 计数为0，执行空操作(noop)
    - 计数大于0，协程进入阻塞状态

- 如果一个 Add(delta) 或者 Done() 调用将 计数更改成一个负数，会生成一个panic

- wg是一个结构体，作为参数传递时需要传递指针

- 示例

  ```go
  package main
  
  import (
      "sync"
  )
  
  // 全局变量
  var counter int
  
  func main() {
      var wg sync.WaitGroup  //
      // var l sync.Mutex //不加锁会导致多次执行结果不同
      for i := 0; i < 1000; i++ {
          wg.Add(1)
          go func() {
          	defer wg.Done()
              // l.Lock()
              counter++
              // l.Unlock()
          }()
      }
  
      wg.Wait()
      println(counter)
  }
  
  ```

  