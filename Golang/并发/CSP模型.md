# 什么是CSP

CSP模型用来描述两个独立的并发实体通过共享的通讯channel管道进行通信的并发模型。

golang借用了CSP模型的一些概念如：实体 process，通道 channel，为之实现并发进行了理论支持，实际上并没有完全实现CSP模型的所有理论。process是在go语言上的表现就是goroutine，是实际上并发执行的实体，每个实体之间是通过channel通讯来实现数据共享。





channel是被单独创建并且可以在进程之间传递，它的通信模式类似于boss-worker模式，一个实体通过将消息发送到channel中，然后又监听这个channel的实体处理，两个实体之间是匿名的，实现原理上其实是一个阻塞的消息队列。

具体可以分为：有/无缓存channel，只读channel，只写channel，双向channel