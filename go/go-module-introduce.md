# Go Module详细使用教程

## **Golang环境变量**

### GOROOT

go的安装路径 在`~/.bash_profile`中添加下面语句配置`GOROOT`环境变量  `export GOROOT=/usr/local/go` 。要执行go命令和go工具, 就要配置go的可执行文件的路径: `export $PATH:$GOROOT/bin`&#x20;

> 注：$PATH windows用`;`符号分割, mac和类unix用`:`符号分割

### GOPATH

go的工作路径 可以在自己的用户目录下面创建一个目录, 如 gopath&#x20;

```bash
cd ~ 
mkdir gopath
```

&#x20;在`~/.bash_profile`中添加如下语句: `export GOPATH=/Users/username/gopath` 。

不要把`GOPATH`设置成go的安装路径, `GOPATH`下主要包含三个目录：`bin` ， `pkg` ，`src`&#x20;

> 注：Go 1.8 版本之前，GOPATH 环境变量默认是空的；1.8版本之后，默认路径是：`$HOME/go`

* src: 存放源代码（比如：.go .c .h .s等）
* pkg: 编译后生成的文件（比如：.a）
* bin: 编译后生成的可执行文件, 为了方便，可以把此目录加入到 `$PATH` 变量中，如果有多个gopath，那么使用`${GOPATH//://bin:}/bin`添加所有的bin目录

## **Go Modules**

`go modules`是 golang 1.11引入的新特性。模块是相关Go包的集合。modules是源代码交换和版本控制的单元。go命令直接支持使用modules，包括记录和解析对其他模块的依赖性。modules替换旧的基于GOPATH的方法来指定在给定构建中使用哪些源文件。

### GO111MODULE

有三个值：`off`、`on` 和 `auto` (默认值) 。在使用模块的时候`GOPATH` 是无意义的，不过它还是会把下载的依赖储存在 `$GOPATH/src/mod` 中，也会把 `go install` 的结果放在 `$GOPATH/bin` 中。

* `GO111MODULE=off`，无模块支持，go 会从 GOPATH 和 vendor 文件夹寻找包
* `GO111MODULE=on`，模块支持，go 会忽略 GOPATH 和 vendor 文件夹，只根据 `go.mod` 下载依赖
* `GO111MODULE=auto`，在 `$GOPATH/src` 外面且根目录有 `go.mod` 文件时，开启模块支持

### GOPROXY

由于中国政府的网络监管系统，Go 生态系统中有着许多中国 Gopher 们无法获取的模块，比如最著名的 `golang.org/x/...`。并且在中国大陆从 GitHub 获取模块的速度也有点慢。因此需要配置GOPROXY来加速Module依赖下载，这里使用goproxy.cn代理，详细介绍：传送门&#x20;

> 注: 推荐将 GO111MODULE 设置为on 而不是auto

* Go 1.13及以上版本 `go env -w GOPROXY=https://goproxy.cn,direct`
* Go 1.13以下的版本 `export GOPROXY=https://goproxy.cn`

## **Go mod**

Golang 1.11 版本引入的 go mod ，其思想类似maven：摒弃vendor和GOPATH，拥抱本地库。从 Go 1.11 开始，Go 允许在 `$GOPATH/src` 外的任何目录下使用 go.mod 创建项目。在`$GOPATH/src`中，为了兼容性，Go 命令仍然在旧的 GOPATH 模式下运行。从 Go 1.13 开始，Module模式将成为默认模式。

### go mod 命令&#x20;

```bash
> go help mod
Go mod provides access to operations on modules.

Note that support for modules is built into all the go commands,
not just 'go mod'. For example, day-to-day adding, removing, upgrading,
and downgrading of dependencies should be done using 'go get'.
See 'go help modules' for an overview of module functionality.

Usage:

    go mod <command> [arguments]

The commands are:

    download    download modules to local cache
    edit        edit go.mod from tools or scripts
    graph       print module requirement graph
    init        initialize new module in current directory
    tidy        add missing and remove unused modules
    vendor      make vendored copy of dependencies
    verify      verify dependencies have expected content
    why         explain why packages or modules are needed

Use "go help mod <command>" for more information about a command.
```

### go.mod

有四种指令：module，require，exclude，replace。

* `module`：模块名称
* `require`：依赖包列表以及版本
* `exclude`：禁止依赖包列表（仅在当前模块为主模块时生效）
* `replace`：替换依赖包列表 （仅在当前模块为主模块时生效）

### go build -mod 编译模式选择

开启 `GO111MODULE=on` 后，go build 将使用 mod 模式寻找依赖包进行编译，GOPATH/src 目录下的依赖将是无效的。其中 go build 可以携带 -mod 的 flag 用于选择不同模式的 mod 编译. 包括 -mod=vendor、-mod=mod、-mod=readonly。

默认情况下，使用的是 -mod=readonly 模式。但在 go 1.14及以上的版本中，如果目录中出现了 vendor 目录，将默认使用 -mod=vendor 模式进行编译。 三种 go build 的 flag 如下：

* **-mod=readonly** 只读模式，如果待引入的 package 不在 go.mod 文件的列表中。不会修改 go.mod ，而是报错。 此外，若模块的 checksum 不在 go.sum 中也会报错。这种模式可以在编译时候避免隐式修改 go.mod。
* **-mod=vendor** 模式下。 将使用工程的 vendor 目录下的 package 而不是 mod cache( GOPATH/pkg/mod) 目录。该模式下编译，将不会检查 go.mod 文件下的包版本。但是会检查 vendor 目录下的 modules.txt(由 go mod vendor 生成)。在 go.1.14 及更高版本，若存在 vendor 目录，将优先使用 vendor 模式。
* **-mod=mod** 模式下，将使用 module cache，即使存在 vendor 目录，也会使用 GOPATH/pkg/mod 下的package，若 package 不存在，将自动下载指定版本的 package。

![](<../.gitbook/assets/image (71).png>)

### 新项目

你可以在`GOPATH`之外创建新的项目。 `go mod init packagename`可以创建一个空的`go.mod`,然后你可以在其中增加`require github.com/smallnest/rpcx latest`依赖，或者像上面一样让go自动发现和维护。

`go mod download`可以下载所需要的依赖，但是依赖并不是下载到`$GOPATH`中，而是`$GOPATH/pkg/mod`中，多个项目可以共享缓存的module。

### 老项目

&#x20;假设你已经有了一个go 项目， 比如在`$GOPATH/github.com/smallnest/rpcx`下， 你可以使用`go mod init github.com/smallnest/rpcx`在这个文件夹下创建一个空的`go.mod` (只有第一行 `module github.com/smallnest/rpcx`)。 然后你可以通过 `go get ./...`让它查找依赖，并记录在`go.mod`文件中(你还可以指定 `-tags`,这样可以把tags的依赖都查找到)。 通过`go mod tidy`也可以用来为`go.mod`增加丢失的依赖，删除不需要的依赖，但是我不确定它怎么处理`tags`。 执行上面的命令会把`go.mod`的`latest`版本换成实际的最新的版本，并且会生成一个`go.sum`记录每个依赖库的版本和哈希值。

## **创建一个新项目**

### 初始化

在`GOPATH 目录之外`新建一个目录，并使用`go mod init` 初始化生成`go.mod` 文件

```bash
➜  ~ mkdir hello
➜  ~ cd hello
➜  hello go mod init hello
go: creating new go.mod: module hello
➜  hello ls
go.mod
➜  hello cat go.mod
module hello

go 1.12
```

> go.mod文件一旦创建后，它的内容将会被go toolchain全面掌控。go toolchain会在各类命令执行时，比如go get、go build、go mod等修改和维护go.mod文件。

go.mod 提供了`module`, `require`、`replace`和`exclude` 四个命令

* `module` 语句指定包的名字（路径）
* `require` 语句指定的依赖项模块
* `replace` 语句可以替换依赖项模块
* `exclude` 语句可以忽略依赖项模块

### 添加依赖

新建一个 server.go 文件，写入以下代码：

```go
package main

import (
	"net/http"
	
	"github.com/labstack/echo"
)

func main() {
	e := echo.New()
	e.GET("/", func(c echo.Context) error {
		return c.String(http.StatusOK, "Hello, World!")
	})
	e.Logger.Fatal(e.Start(":1323"))
}
```

执行 `go run server.go` 运行代码会发现 go mod 会自动查找依赖自动下载：

```bash
$ go run server.go
go: finding github.com/labstack/echo v3.3.10+incompatible
go: downloading github.com/labstack/echo v3.3.10+incompatible
go: extracting github.com/labstack/echo v3.3.10+incompatible
go: finding github.com/labstack/gommon/color latest
go: finding github.com/labstack/gommon/log latest
go: finding github.com/labstack/gommon v0.2.8
# 此处省略很多行
...

   ____    __
  / __/___/ /  ___
 / _// __/ _ \/ _ \
/___/\__/_//_/\___/ v3.3.10-dev
High performance, minimalist Go web framework
https://echo.labstack.com
____________________________________O/_______
                                    O\
⇨ http server started on [::]:1323
```

现在查看go.mod 内容：

```go
$ cat go.mod

module hello

go 1.12

require (
	github.com/labstack/echo v3.3.10+incompatible // indirect
	github.com/labstack/gommon v0.2.8 // indirect
	github.com/mattn/go-colorable v0.1.1 // indirect
	github.com/mattn/go-isatty v0.0.7 // indirect
	github.com/valyala/fasttemplate v1.0.0 // indirect
	golang.org/x/crypto v0.0.0-20190313024323-a1f597ede03a // indirect
)
```

go module 安装 package 的原則是先拉最新的 release tag，若无tag则拉最新的commit。go 会自动生成一个 go.sum 文件来记录 dependency tree：

```go
$ cat go.sum
github.com/labstack/echo v3.3.10+incompatible h1:pGRcYk231ExFAyoAjAfD85kQzRJCRI8bbnE7CX5OEgg=
github.com/labstack/echo v3.3.10+incompatible/go.mod h1:0INS7j/VjnFxD4E2wkz67b8cVwCLbBmJyDaka6Cmk1s=
github.com/labstack/gommon v0.2.8 h1:JvRqmeZcfrHC5u6uVleB4NxxNbzx6gpbJiQknDbKQu0=
github.com/labstack/gommon v0.2.8/go.mod h1:/tj9csK2iPSBvn+3NLM9e52usepMtrd5ilFYA+wQNJ4=
github.com/mattn/go-colorable v0.1.1 h1:G1f5SKeVxmagw/IyvzvtZE4Gybcc4Tr1tf7I8z0XgOg=
github.com/mattn/go-colorable v0.1.1/go.mod h1:FuOcm+DKB9mbwrcAfNl7/TZVBZ6rcnceauSikq3lYCQ=
... 省略很多行
```

再次执行脚本 `go run server.go` 发现跳过了检查并安装依赖的步骤。

### 升级依赖

可以使用命令 `go list -m -u all` 来检查可以升级的package，使用`go get -u need-upgrade-package` 升级后会将新的依赖版本更新到 go.mod。也可以使用 `go get -u` 升级所有依赖。go get 升级：

* 运行 go get -u 将会升级到最新的次要版本或者修订版本(x.y.z, z是修订版本号， y是次要版本号)
* 运行 go get -u=patch 将会升级到最新的修订版本
* 运行 go get package@version 将会升级到指定的版本号version
* 运行go get如果有版本的更改，那么go.mod文件也会更改

## **改造现有项目(helloword)**

### 现有状态

项目目录为：

```bash
$ tree
.
├── api
│   └── apis.go
└── server.go

1 directory, 2 files
```

server.go 源码为：

```go
package main

import (
    api "./api"  // 这里使用的是相对路径
    "github.com/labstack/echo"
)

func main() {
    e := echo.New()
    e.GET("/", api.HelloWorld)
    e.Logger.Fatal(e.Start(":1323"))
}
```

api/apis.go 源码为：

```go
package api

import (
    "net/http"

    "github.com/labstack/echo"
)

func HelloWorld(c echo.Context) error {
    return c.JSON(http.StatusOK, "hello world")
}
```

### 初始化

> export GO111MODULE=on 先设置环境变量开启Module功能

使用 `go mod init ***` 初始化go.mod

```
$ go mod init helloworld
go: creating new go.mod: module helloworld
```

### 添加依赖

运行 `go run server.go`

```
go: finding github.com/labstack/gommon/color latest
go: finding github.com/labstack/gommon/log latest
go: finding golang.org/x/crypto/acme/autocert latest
go: finding golang.org/x/crypto/acme latest
go: finding golang.org/x/crypto latest
build command-line-arguments: cannot find module for path _/home/gs/helloworld/api
```

首先还是会查找并下载安装依赖，然后运行脚本 `server.go`，这里会抛出一个错误：

```
build command-line-arguments: cannot find module for path _/home/gs/helloworld/api
```

但是`go.mod` 已经更新：

```go
$ cat go.mod
module helloworld

go 1.12

require (
        github.com/labstack/echo v3.3.10+incompatible // indirect
        github.com/labstack/gommon v0.2.8 // indirect
        github.com/mattn/go-colorable v0.1.1 // indirect
        github.com/mattn/go-isatty v0.0.7 // indirect
        github.com/valyala/fasttemplate v1.0.0 // indirect
        golang.org/x/crypto v0.0.0-20190313024323-a1f597ede03a // indirect
)
```

### **解决故障**

那为什么会抛出这个错误呢？

这是因为 server.go 中使用 internal package 的方法跟以前已经不同了，由于 go.mod会扫描同工作目录下所有 package 并且`变更引入方法`，必须将 helloworld当成路径的前缀，也就是需要写成 `import helloworld/api`，以往 GOPATH/dep 模式允许的 import ./api 已经失效，详情可以查看这个 issue。

更新旧的package import 方式， 所以server.go 需要改写成：

```go
package main

import (
    api "helloworld/api"  // 这是更新后的引入方法
    "github.com/labstack/echo"
)

func main() {
    e := echo.New()
    e.GET("/", api.HelloWorld)
    e.Logger.Fatal(e.Start(":1323"))
}
```

> `一个小坑`：开始在golang1.11 下使用go mod 遇到过 `go build github.com/valyala/fasttemplate: module requires go 1.12` 这种错误，遇到类似这种需要升级到1.12 的问题，直接升级golang1.12 就好了。幸亏是在1.12 发布后才尝试的`go mod` ?‍♂️



到这里就和新创建一个项目没什么区别了

## **总结**

Go Module是Go依赖管理的未来。从1.11之后开始支持该功能，随着Go依赖管理的功能增强，以后再也不用被现在的包管理犯难了。



## References

* [原文 Go Module详细使用教程，包管理不在难](https://cloud.tencent.com/developer/article/1593734), [咻咻ing](https://cloud.tencent.com/developer/user/5974198)
* [Go Modules使用教程](https://segmentfault.com/a/1190000016703769), [andyidea](https://segmentfault.com/u/andyidea)
* [Go包管理--GOPATH、vendor、go mod机制](https://www.mdeditor.tw/pl/ggmD)&#x20;
