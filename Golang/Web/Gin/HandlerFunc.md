# HandlerFunc是什么

- 一种函数类型

- 实现了Handler接口的ServerHttp方法，是Handler的具体类

- 是一种适配器，允许将一个普通函数作为http的处理器

- 源码：

  ```go
  // The HandlerFunc type is an adapter to allow the use of
  // ordinary functions as HTTP handlers. If f is a function
  // with the appropriate signature, HandlerFunc(f) is a
  // Handler that calls f.
  type HandlerFunc func(ResponseWriter, *Request)
  
  // ServeHTTP calls f(w, r).
  func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
      f(w, r)
  }
  
  ```

  

# gin.HandlerFunc





# http.HandlerFunc



