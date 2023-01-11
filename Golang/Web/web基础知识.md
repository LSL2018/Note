# 路由

## route和router

- 路由（route）是URL到函数的映射，例如：

  ```
  /users        ->  getAllUsers()
  /users/count  ->  getUsersCount()
  ```

  上述就是两条路由，当访问 /users 的时候，会执行 getAllUsers() 函数；当访问 /users/count 的时候，会执行 getUsersCount() 函数

- 路由器（router） 管理了一组 route：当接收到一个URL之后，去路由映射表中查找相应的函数，这个过程是由 router 来处理的。

- 在 router 匹配 route 的过程中，不仅会根据URL来匹配，还会根据请求的方法（GET，POST……等）来看是否匹配。





# Content-Type





# Cookie





# X-Forwarded-For和X-Real-Ip

