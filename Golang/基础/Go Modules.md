# 前提

​				流行的现代编程语言一般都提供依赖库管理工具，如 Java 的 [Maven](https://so.csdn.net/so/search?q=Maven&spm=1001.2101.3001.7020) 、Python 的 PIP、Node.js 的 NPM 和 Rust 的 Cargo 等。Go 最为一门新生代语言，自然也有其自己的库管理方式。

# 演变过程

## GOPATH

- 在 Go 1.5 之前，Go 的依赖管理使用的是 go get，执行命令后会拉取代码放入 $GOPATH/src 下面；

- 是作为 GOPATH 下全局的依赖；

- 不能进行版本控制，以及隔离项目的包依赖。

- 从Go 1.8版本开始，Go开发包在安装完成后会为gopath设置一个默认目录

- gopath模式下$GOPATH目录结构：

  ​    -- bin：存放编译后生成的二进制可执行文件

  ​    -- pkg：存放编译后生成的.a文件

  ​    -- src：存放项目的源代码，可以是自己写的代码，也可以是go get下载的包



## Go vendor

- Go 1.5 版本推出了 vendor 机制，需要手动设置环境变量 GO15VENDOREXPERIMENT= 1，Go 编译器才能启用；从 Go1.6 起，默认开启 vendor 机制。
- 机制：每个项目的根目录下可以有一个 vendor 目录，里面存放了该项目的依赖的 package；go build 的时候会先去 vendor 目录查找依赖，如果没有找到会再去 GOPATH 目录下查找。
-  缺点：对外部依赖的第三方包的版本管理：使用 go get -u 更新第三方包，默认的是将工程的默认分支的最新版本拉取到本地，但并不能指定第三方包的版本。而在实际包升级过程中，如果发现新版本有问题，则不能很快回退。



## Go Modules

- Go 1.11 开始，Go 官方推出了 Go Modules，需要显示通过设置一个环境变量 GO111MODULE=on；在 Go 1.12 正式推出后，Go Modules 成为默认的依赖管理方式；
- 在项目根文件下有：
  -  go.mod ：定义 module path 和依赖库的版本；
  -  go.sum ：包含特定依赖包的版本内容的散列哈希值。



# Go Modules 

## go.mod

示例：

```go
module github.com/dablelv/go-huge-util

go 1.17

replace github.com/coreos/bbolt => ../r

require (
	github.com/cenk/backoff v2.2.1+incompatible
	github.com/edwingeng/doublejump v0.0.0-20200330080233-e4ea8bd1cbed
	github.com/spf13/cast v1.4.1
)

require (
	github.com/davecgh/go-spew v1.1.1 // indirect
)

exclude (
	go.etcd.io/etcd/client/v2 v2.305.0-rc.0
)

retract (
    v1.0.0 // 废弃的版本，请使用v1.1.0
)
```

解析：

- 第一行：是module path，一般采用“仓库+module name”的方式定义
- 第二行：并不是当前使用的 Go 版本，而是指名项目代码所需要的 Go 的最低版本；非必须
- require：列出了项目所需要的各个依赖库以及它们的版本
  - 类似v1.4.1这样的是正规版本
  - v0.0.0-20200330080233-e4ea8bd1cbed是伪版本号，是 go module 为它生成的一个类似符合语义化版本 2.0.0 版本，实际这个库并没有发布这个版本
- indirect 注释：
  - 从 go 1.17 开始，加了indirect注释 的 module 将被放在单独 require 块
  - 以下情况会加 indirect 注释：
    - 当前项目依赖 A，但是 A 的go.mod 遗漏了 B，那么就会在当前项目的 go.mod 中补充 B，加 indirect 注释
    - 当前项目依赖 A，但是 A 没有 go.mod，同样就会在当前项目的 go.mod 中补充 B，加 indirect 注释
    - 当前项目依赖 A，A 又依赖 B。当对 A 降级的时候，降级的 A 不再依赖 B，这个时候 B 就标记 indirect 注释。我们可以执行 go mod tidy 来清理不依赖的 module
- incompatible：加了incompatible后缀，表示该库采用了 go.mod 的管理， major 版本已经 >=2 了，但是该库的 module path 中依然没有添加 v2、v3 这样的后缀，所以可以引用，但不符合 Go 的 module 管理规范
- exclude：跳过某个依赖库的某个版本
- replace：解决一些错误的依赖库的引用或者调试依赖库
- retract：宣布撤回库的某个版本；和 exclude 的区别是 retract 是这个库的 owner 定义的， 而 exclude 是库的使用者在自己的 go.mod 中定义的

## go.sum

- 记录了所有依赖的 module 的校验信息，以防下载的依赖被恶意篡改，主要用于安全校验

- 每行格式：

  ```go
  <module> <version> <hash>
  <module> <version>/go.mod <hash>
  ```

  - module 是依赖的路径。
  - version 是依赖的版本号；如果 version 后面跟/go.mod表示对哈希值是 module 的 go.mod 文件；否则，哈希值是 module 的.zip文件。
  - hash 是以h1:开头的字符串，表示生成 checksum 的算法是第一版的 HASH 算法（SHA256）。如果将来在 SHA-256 中发现漏洞，将添加对另一种算法的支持，可能会命名为 h2



# Go mod命令

```shell
#在当前目录中初始化和创建一个新的go.mod文件
go mod init

#下载 go.mod 文件中指明的所有依赖，使用此命令来下载指定的模块
go mod download

#整理现有的依赖，使用此命令来下载指定的模块，并删除已经不用的模块
go mod tidy

#查看现有的依赖结构，生成项目所有依赖的报告，但可读性太差，图形化更方便
go mod graph

#编辑 go.mod 文件，之后通过 download 或 edit 进行下载
go mod edit

#导出项目所有的依赖到vendor目录，从mod中拷贝到项目的vendor目录下
go mod vendor

#校验一个模块是否被篡改过，查询某个常见的模块出错是否已被篡改
go mod verify

#查看为什么需要依赖某模块，查询某个不常见的模块是否是哪个模块的引用
go mod why


```

