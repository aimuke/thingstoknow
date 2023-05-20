# Go-项目结构和代码组织（go 开发框架）



## 开源界优秀项目的结构示例

因为最新的 Go 版本已经[使用](https://www.finclip.com/news/tags-92.html) module 作为版本依赖，所以，所有项目的 vendor 我都忽略，建议直接使用 module 来管理依赖，而且较好的解决某些库国内访问不了的问题，参考：https://studygolang.com/topics/8737

### Docker

https://github.com/moby/moby

```
├── api      // 存放对外公开的 API 规则
├── builder  // 存放构建脚本等
├── cli      // 命令行的主要逻辑
├── cmd      // 存放可执行程序，main 包放这个目录中
├── contrib  // 存放一些有用的脚本或文件，但不是项目的核心部分
├── docs    // 存放文档
├── internal // 只在本项目使用的包（私有）
├── pkg     // 本项目以及其他项目可以使用的包（公有）
├── plugin  // 提供插件功能
```

### Kubernetes

https://github.com/kubernetes/kubernetes

```
├── api
├── build  // 存放构建脚本等
├── cmd
├── docs
├── pkg
├── plugin
├── test    // 单元测试之外的测试程序、测试数据
├── third_party // 经过修改的第三方的代码
```

### Gogs

https://github.com/gogs/gogs

```
├── cmd
├── conf    // 对配置进行解析
├── docker  // 存放 docker 脚本
├── models  // MVC 中的 model
├── pkg
├── public  // 静态公共资源，实际项目会将其存入 CDN
├── routes  // 路由
├── scripts // 脚本文件
├── templates // 存放模板文件
```



### influxdb

https://github.com/influxdata/influxdb

```
├── cmd
├── docker
├── docs
├── http // 存放 HTTP Handler 等，相当于 MVC 的 Controller
├── internal
├── models
├── pkg
├── scripts
```



## 开源项目小结

总体上，这些优秀开源项目，没有统一一致的目录结构方式，但大体上，有一些通用的地方，这就有了\*\* https://github.com/golang-standards/project-layout \*\*这个项目。

标准 Go 项目布局（结构）

https://github.com/golang-standards/project-layout 项目总结了 Go 项目的布局，我们一起看看这些主要的目录。

/cmd

该目录用于存放 Go 项目的入口，即 main.main。一般来说，我们应该在 cmd 目录下创建子目录，子目录名称代表可执行程序的名称。上面列出的优秀开源项目基本上遵循了这一规则。

事实上，Go 语言本身，以及 github.com/golang/tools 都采用了 cmd 及其子目录的形式，所以咱们的项目没有理由不使用。

一般来说，该目录中的代码应该尽可能少。

/internal

这是 Go 包的一个特性，放在该包中的代码，表明只希望项目内部使用，是项目或库私有的，其他项目或库不能使用。

/pkg

该包可以和 internal 对应，是公开的。一般来说，放在该包的代码应该和具体业务无关，方便本项目和其他项目重用。当你决定将代码放入该包时，你应该对其负责，因为别人很可能使用它。

因为 GOPATH 中有一个目录就是 pkg，所以，社区有些人对该目录不太能接受。但不管怎么样，开源界有很多优秀项目在使用它，这里有一些使用它的项目列表：

https://github.com/golang-standards/project-layout/blob/master/pkg/README.md

/api

该目录用来存放 OpenAPI/Swagger 规则说明, JSON 格式定义, 协议定义文件等。也有可能用来存放具体的对外公开 API，比如 Docker：https://github.com/moby/moby/tree/master/api/server 。

/init

存放随着系统自动启动脚本，如：systemd, upstart, sysv；或者通过 supervisor 进行进程管理的脚本。

/scripts

存放 build、install、analysis 等操作脚本。这些脚本使得项目根目录的 Makefile 很简洁。

/build

该目录用于存放打包和持续集成相关脚本。

/test

一般用来存放除单元测试、基准测试之外的测试，比如集成测试、测试数据等。

Go 语言源码仓库中就有 test 目录。

/docs

存放设计和用户文档

/tools

存放项目的支持工具。

/third\_party

从第三代码包抽取过来的。根据官方建议，包名不应该有 \_，所以本人不建议使用。真有这样的需要，考虑命名为 thirdparty。



## 原文地址 [https://www.finclip.com/news/f/2066.html](https://www.finclip.com/news/f/2066.html)
