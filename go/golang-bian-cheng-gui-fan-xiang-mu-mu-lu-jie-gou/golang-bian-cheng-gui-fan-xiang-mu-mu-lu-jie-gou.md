# golang 编程规范 - 项目目录结构

### 目录结构 <a href="#mu-lu-jie-gou" id="mu-lu-jie-gou"></a>

项目的**目录结构**通常也是**门面**，内行人通过目录结构基本就能看出**开发者是否有经验**。

**Go 官网并没有给出一个目录结构的标准模板**，但是 [golang-standards](https://github.com/golang-standards/project-layout) 倒是给出了一个，目录结构如下：

```
```

不过，笔者在实践中发现 [golang-standards](https://github.com/golang-standards/project-layout) 的目录结构也存在一些问题。

笔者将这些问题以**注解的方式**写在下文的具体目录讲解中，欢迎大家一起讨论。

### Go 目录 <a href="#go-mu-lu" id="go-mu-lu"></a>

#### cmd <a href="#cmd" id="cmd"></a>

当前项目的**可执行文件**。`cmd` 目录下的每一个**子目录名称都应该匹配可执行文件**。比如果我们的项目是一个 `grpc` 服务，在 /cmd/**myapp**/main.go 中就包含了启动服务进程的代码，编译后生成的可执行文件就是 **myapp**。

不要在 `/cmd` 目录中放置太多的代码，我们应该将**公有代码**放置到 `/pkg` 中，将**私有代码**放置到 `/internal` 中并在 `/cmd` 中引入这些包，**保证 main 函数中的代码尽可能简单和少**。

例子：

* [https://github.com/heptio/ark/tree/master/cmd](https://github.com/heptio/ark/tree/master/cmd)
* [https://github.com/moby/moby/tree/master/cmd](https://github.com/moby/moby/tree/master/cmd)
* [https://github.com/prometheus/prometheus/tree/master/cmd](https://github.com/prometheus/prometheus/tree/master/cmd)
* [https://github.com/influxdata/influxdb/tree/master/cmd](https://github.com/influxdata/influxdb/tree/master/cmd)
* [https://github.com/kubernetes/kubernetes/tree/master/cmd](https://github.com/kubernetes/kubernetes/tree/master/cmd)
* [https://github.com/dapr/dapr/tree/master/cmd](https://github.com/dapr/dapr/tree/master/cmd)
* [https://github.com/ethereum/go-ethereum/tree/master/cmd](https://github.com/ethereum/go-ethereum/tree/master/cmd)

> 注：`cmd` 目录存在有一个前提，那就是项目有**多个可执行文件**，如果你的项目是**微服务**，那么通常是**只有一个可执行文件**的。这时，笔者建议大家直接将 `main.go` 放在项目根目录下，而取消 `cmd` 目录。

#### internal <a href="#internal" id="internal"></a>

**私有的**应用程序代码库。这些是不希望被其他人导入的代码。请注意：这种模式是 Go **编译器强制执行**的。有关更多细节，请参阅 Go 1.4 的 [release notes](https://golang.org/doc/go1.4#internalpackages)。并且，在项目的目录树中的**任意位置都可以有 internal 目录**，而不仅仅是在顶级目录中。

可以在内部代码包中添加一些额外的结构，来分隔共享和非共享的内部代码。这不是必选项（尤其是在小项目中），但是有一个直观的包用途是很棒的。比如：应用程序代码放在 `/internal/app` 目录（如，`internal/app/myapp`），而应用程序的共享代码放在 `/internal/pkg` 目录（如，`internal/pkg/myprivlib`）中。

> 注：`internal` 目录的问题与 `cmd` 类似，如果你的项目是**微服务**，那么建议大家可以改成 `internal/myapp` 。

#### pkg <a href="#pkg" id="pkg"></a>

**外部应用程**序可以使用的库代码（如，`/pkg/mypubliclib`）。其他项目将会导入这些库来保证项目可以正常运行，所以在将代码放在这里前，一定要三四而行。请注意，`internal` 目录是一个更好的选择来确保项目私有代码不会被其他人导入，因为这是 Go 强制执行的。使用 `/pkg` 目录来明确表示代码可以被其他人安全的导入仍然是一个好方式。Travis Jeffery 撰写的关于 [I’ll take pkg over internal](https://travisjeffery.com/b/2019/11/i-ll-take-pkg-over-internal/) 文章很好地概述了 `pkg` 和 `inernal` 目录以及何时使用它们。

当根目录包含大量非 Go 组件和目录时，这也是一种将 Go 代码分组到一个位置的方法，从而使运行各种 Go 工具更加容易（在如下的文章中都有提到：[2018 年 GopherCon Best Practices for Industrial Programming](https://www.youtube.com/watch?v=PTE4VJIdHPg)，[GopherCon 2018: Kat Zien - How Do You Structure Your Go Apps](https://www.youtube.com/watch?v=oL6JBUk6tj0)，[Golab 2018 Massimiliano Pippi - Project layout patterns in Go](https://www.youtube.com/watch?v=3gQa1LWwuzk)。

`/pkg` 在许多开源项目中都使用了，但**未被普遍接受，并且 Go 社区中的某些人不推荐这样做**。

如果**项目确实很小**并且嵌套的层次并不会带来多少价值（除非你就是想用它），那么就不要使用它。**但当项目变得很大，并且根目录中包含的内容相当繁杂**（尤其是有很多非 Go 的组件）时，可以考虑使用 `/pkg`。

例子：

* [https://github.com/prometheus/prometheus/tree/master/pkg](https://github.com/prometheus/prometheus/tree/master/pkg)
* [https://github.com/jaegertracing/jaeger/tree/master/pkg](https://github.com/jaegertracing/jaeger/tree/master/pkg)
* [https://github.com/istio/istio/tree/master/pkg](https://github.com/istio/istio/tree/master/pkg)
* [https://github.com/GoogleContainerTools/kaniko/tree/master/pkg](https://github.com/GoogleContainerTools/kaniko/tree/master/pkg)
* [https://github.com/google/gvisor/tree/master/pkg](https://github.com/google/gvisor/tree/master/pkg)
* [https://github.com/google/syzkaller/tree/master/pkg](https://github.com/google/syzkaller/tree/master/pkg)
* [https://github.com/perkeep/perkeep/tree/master/pkg](https://github.com/perkeep/perkeep/tree/master/pkg)
* [https://github.com/minio/minio/tree/master/pkg](https://github.com/minio/minio/tree/master/pkg)
* [https://github.com/heptio/ark/tree/master/pkg](https://github.com/heptio/ark/tree/master/pkg)
* [https://github.com/argoproj/argo/tree/master/pkg](https://github.com/argoproj/argo/tree/master/pkg)
* [https://github.com/heptio/sonobuoy/tree/master/pkg](https://github.com/heptio/sonobuoy/tree/master/pkg)
* [https://github.com/helm/helm/tree/master/pkg](https://github.com/helm/helm/tree/master/pkg)
* [https://github.com/kubernetes/kubernetes/tree/master/pkg](https://github.com/kubernetes/kubernetes/tree/master/pkg)
* [https://github.com/kubernetes/kops/tree/master/pkg](https://github.com/kubernetes/kops/tree/master/pkg)
* [https://github.com/moby/moby/tree/master/pkg](https://github.com/moby/moby/tree/master/pkg)
* [https://github.com/grafana/grafana/tree/master/pkg](https://github.com/grafana/grafana/tree/master/pkg)
* [https://github.com/influxdata/influxdb/tree/master/pkg](https://github.com/influxdata/influxdb/tree/master/pkg)
* [https://github.com/cockroachdb/cockroach/tree/master/pkg](https://github.com/cockroachdb/cockroach/tree/master/pkg)
* [https://github.com/derekparker/delve/tree/master/pkg](https://github.com/derekparker/delve/tree/master/pkg)
* [https://github.com/etcd-io/etcd/tree/master/pkg](https://github.com/etcd-io/etcd/tree/master/pkg)
* [https://github.com/oklog/oklog/tree/master/pkg](https://github.com/oklog/oklog/tree/master/pkg)
* [https://github.com/flynn/flynn/tree/master/pkg](https://github.com/flynn/flynn/tree/master/pkg)
* [https://github.com/jesseduffield/lazygit/tree/master/pkg](https://github.com/jesseduffield/lazygit/tree/master/pkg)
* [https://github.com/gopasspw/gopass/tree/master/pkg](https://github.com/gopasspw/gopass/tree/master/pkg)
* [https://github.com/sosedoff/pgweb/tree/master/pkg](https://github.com/sosedoff/pgweb/tree/master/pkg)
* [https://github.com/GoogleContainerTools/skaffold/tree/master/pkg](https://github.com/GoogleContainerTools/skaffold/tree/master/pkg)
* [https://github.com/knative/serving/tree/master/pkg](https://github.com/knative/serving/tree/master/pkg)
* [https://github.com/grafana/loki/tree/master/pkg](https://github.com/grafana/loki/tree/master/pkg)
* [https://github.com/bloomberg/goldpinger/tree/master/pkg](https://github.com/bloomberg/goldpinger/tree/master/pkg)
* [https://github.com/Ne0nd0g/merlin/tree/master/pkg](https://github.com/Ne0nd0g/merlin/tree/master/pkg)
* [https://github.com/jenkins-x/jx/tree/master/pkg](https://github.com/jenkins-x/jx/tree/master/pkg)
* [https://github.com/DataDog/datadog-agent/tree/master/pkg](https://github.com/DataDog/datadog-agent/tree/master/pkg)
* [https://github.com/dapr/dapr/tree/master/pkg](https://github.com/dapr/dapr/tree/master/pkg)
* [https://github.com/cortexproject/cortex/tree/master/pkg](https://github.com/cortexproject/cortex/tree/master/pkg)
* [https://github.com/dexidp/dex/tree/master/pkg](https://github.com/dexidp/dex/tree/master/pkg)
* [https://github.com/pusher/oauth2\_proxy/tree/master/pkg](https://github.com/pusher/oauth2\_proxy/tree/master/pkg)
* [https://github.com/pdfcpu/pdfcpu/tree/master/pkg](https://github.com/pdfcpu/pdfcpu/tree/master/pkg)
* [https://github.com/weaveworks/kured/tree/master/pkg](https://github.com/weaveworks/kured/tree/master/pkg)
* [https://github.com/weaveworks/footloose/tree/master/pkg](https://github.com/weaveworks/footloose/tree/master/pkg)
* [https://github.com/weaveworks/ignite/tree/master/pkg](https://github.com/weaveworks/ignite/tree/master/pkg)
* [https://github.com/tmrts/boilr/tree/master/pkg](https://github.com/tmrts/boilr/tree/master/pkg)
* [https://github.com/kata-containers/runtime/tree/master/pkg](https://github.com/kata-containers/runtime/tree/master/pkg)
* [https://github.com/okteto/okteto/tree/master/pkg](https://github.com/okteto/okteto/tree/master/pkg)
* [https://github.com/solo-io/squash/tree/master/pkg](https://github.com/solo-io/squash/tree/master/pkg)

> 注：
>
> * 对于 `pkg` 目录，如果是在**微服务下，笔者更建议尽量不使用它**。因为微服务，每个服务都会相对简单，也就是**项目都比较小** pkg 不会带来多大价值。
> * 如果有**公用的代码**，笔者更建议大家将这类代码做成**私有库（go module）**，供其他项目复用，**做了物理隔离，更有利于代码的抽象**。
> * 但是，有一种情况，可以考虑使用 `pkg`，那就是有一类**公用的代码只在有限几个项目中可公用**。比如：在**权限服务**中需要使用到**用户服务**的 `User` 结构体，那这种公用的代码，可以考虑放在用户服务的 `pkg` 中，供权限服务引用。

> 注：在 Go 语言中组织代码的方式还有一种叫”平铺“的，也就是**在根目录下放项目的代码**。这种方式在很多**框架或者库**中非常常见，如果想要引入一个使用 pkg 目录结构的框架时，我们往往需要使用 `github.com/golang/project/pkg/somepkg`，当代码都平铺在项目的根目录时只需要使用 `github.com/golang/project`，很明显地减少了引用依赖包语句的长度。所以，对于一个 Go 语言的**框架或者库，将代码平铺在根目录下也很正常**，但是在一个 Go 语言的**服务中使用这种代码组织方法可能就没有那么合适了**。

#### vendor <a href="#vendor" id="vendor"></a>

应用程序的依赖关系（通过手动或者使用喜欢的依赖管理工具，如新增的内置 Go Modules 特性）。执行 `go mod vendor` 命令将会在项目中创建 `/vendor` 目录，注意，如果使用的不是 Go 1.14 版本，在执行 `go build` 进行编译时，需要添加 `-mod=vendor` 命令行选项，因为它不是默认选项。

构建库文件时，不要提交应用程序依赖项。

请注意，从 1.13 开始，Go 也启动了模块代理特性（使用 [https://proxy.golang.org](https://proxy.golang.org/) 作为默认的模块代理服务器）。点击[这里](https://blog.golang.org/module-mirror-launch)阅读有关它的更多信息，来了解它是否符合所需要求和约束。如果 Go Module 满足需要，那么就不需要 vendor 目录。

国内模块代理功能默认是被墙的，七牛云有维护专门的的[模块代理](https://github.com/goproxy/goproxy.cn/blob/master/README.zh-CN.md)。

> 注：从笔者的实践来看，**Go Module 已经满足需要，不需要 vendor 目录**。

### 服务端应用程序目录 <a href="#fu-wu-duan-ying-yong-cheng-xu-mu-lu" id="fu-wu-duan-ying-yong-cheng-xu-mu-lu"></a>

#### api <a href="#api" id="api"></a>

项目**对外提供和依赖**的 API 文件。比如：**OpenAPI/Swagger specs, JSON schema 文件, protocol 定义文件**等。

比如，[Kubernetes](https://github.com/kubernetes/kubernetes/tree/master/api) 项目的 api 目录结构如下：

```
```

> 注：在 go 中用的比较多的 **gRPC proto 文件，也比较适合放在 api 目录下**。
>
> ```
> ```

### Web 应用程序目录 <a href="#web-ying-yong-cheng-xu-mu-lu" id="web-ying-yong-cheng-xu-mu-lu"></a>

#### web <a href="#web" id="web"></a>

Web 应用程序特定的组件：静态 Web 资源，服务器端模板和单页应用（Single-Page App，SPA）

> 注：如果项目是个**前后端**的，并且是**一个团队开发**的，那么可以将前端项目放在 `web` 目录下，方便**项目管理**、**构建**、**部署**等，如：[https://github.com/appboot/appboot](https://github.com/appboot/appboot)

### 通用应用程序目录 <a href="#tong-yong-ying-yong-cheng-xu-mu-lu" id="tong-yong-ying-yong-cheng-xu-mu-lu"></a>

#### build <a href="#build" id="build"></a>

**打包和持续集成**所需的文件。

* build/ci：存放持续集成的配置和脚本，如果持续集成平台(例如 Travis CI)对配置文件有路径要求，则可将其 link 到指定位置。
* build/package：存放 AMI、Docker、系统包（deb、rpm、pkg）的配置和脚本等。

例子：

* [https://github.com/cockroachdb/cockroach/tree/master/build](https://github.com/cockroachdb/cockroach/tree/master/build)

> 注：笔者觉得将对配置文件有路径要求的 link 到指定位置，太过**为了什么而做什么了**，有点**本末倒置**，这样不但没有太大的价值，而且会**给熟悉配置文件的人感觉疑惑，得不偿失**。

#### configs <a href="#configs" id="configs"></a>

**配置文件**模板或**默认配置**。

#### deployments <a href="#deployments" id="deployments"></a>

IaaS，PaaS，系统和容器编排部署配置和模板（docker-compose，kubernetes/helm，mesos，terraform，bosh）。请注意，在某些存储库中（尤其是使用 kubernetes 部署的应用程序），该目录的名字是 **/deploy**。

> 注：如果是用 kubernetes 部署，建议改成 `deploy`，因为在 kubernetes 领域内更让人熟悉。

#### init <a href="#init" id="init"></a>

系统初始化（`systemd`、`upstart`、`sysv`）和进程管理（`runit`、`supervisord`）配置。

#### scripts <a href="#scripts" id="scripts"></a>

用于执行各种**构建，安装，分析**等操作的脚本。

这些脚本**使根级别的 Makefile 变得更小更简单**（例如，[https://github.com/hashicorp/terraform/blob/master/Makefile](https://github.com/hashicorp/terraform/blob/master/Makefile)）。

例子：

* [https://github.com/kubernetes/helm/tree/master/scripts](https://github.com/kubernetes/helm/tree/master/scripts)
* [https://github.com/cockroachdb/cockroach/tree/master/scripts](https://github.com/cockroachdb/cockroach/tree/master/scripts)
* [https://github.com/hashicorp/terraform/tree/master/scripts](https://github.com/hashicorp/terraform/tree/master/scripts)

#### test <a href="#test" id="test"></a>

**外部测试应用程序和测试数据**。随时根据需要构建 `/test` 目录。对于较大的项目，有一个数据子目录更好一些。例如，如果需要 Go 忽略目录中的内容，则可以使用 `/test/data` 或 `/test/testdata` 这样的目录名字。请注意，Go 还将忽略以“.”或“\_”开头的目录或文件，因此可以更具灵活性的来命名测试数据目录。

例子：

* [https://github.com/openshift/origin/tree/master/test](https://github.com/openshift/origin/tree/master/test) (测试数据在 `/testdata` 子目录)

### 其他目录 <a href="#qi-ta-mu-lu" id="qi-ta-mu-lu"></a>

#### assets <a href="#assets" id="assets"></a>

项目中使用的其他资源（图像、logo 等）。

#### docs <a href="#docs" id="docs"></a>

**设计和用户文档**（除了 godoc 生成的文档）。

例子：

* [https://github.com/gohugoio/hugo/tree/master/docs](https://github.com/gohugoio/hugo/tree/master/docs)
* [https://github.com/openshift/origin/tree/master/docs](https://github.com/openshift/origin/tree/master/docs)
* [https://github.com/dapr/dapr/tree/master/docs](https://github.com/dapr/dapr/tree/master/docs)

#### examples <a href="#examples" id="examples"></a>

应用程序或公共库的示例程序。

例子：

* [https://github.com/nats-io/nats.go/tree/master/examples](https://github.com/nats-io/nats.go/tree/master/examples)
* [https://github.com/docker-slim/docker-slim/tree/master/examples](https://github.com/docker-slim/docker-slim/tree/master/examples)
* [https://github.com/gohugoio/hugo/tree/master/examples](https://github.com/gohugoio/hugo/tree/master/examples)
* [https://github.com/hashicorp/packer/tree/master/examples](https://github.com/hashicorp/packer/tree/master/examples)

#### githooks <a href="#githooks" id="githooks"></a>

Git 钩子。

#### third\_party <a href="#third_party" id="third_party"></a>

外部辅助工具，**fork 的代码**和其他第三方工具（例如：Swagger UI）。

#### tools <a href="#tools" id="tools"></a>

此项目的支持工具。请注意，这些工具可以从 `/pkg` 和 `/internal` 目录导入代码。

例子：

* [https://github.com/istio/istio/tree/master/tools](https://github.com/istio/istio/tree/master/tools)
* [https://github.com/openshift/origin/tree/master/tools](https://github.com/openshift/origin/tree/master/tools)
* [https://github.com/dapr/dapr/tree/master/tools](https://github.com/dapr/dapr/tree/master/tools)

#### website <a href="#website" id="website"></a>

如果不使用 Github pages，则在这里放置项目的网站数据。

例子：

* [https://github.com/hashicorp/vault/tree/master/website](https://github.com/hashicorp/vault/tree/master/website)
* [https://github.com/perkeep/perkeep/tree/master/website](https://github.com/perkeep/perkeep/tree/master/website)

### 不应该出现的目录 <a href="#bu-ying-gai-chu-xian-de-mu-lu" id="bu-ying-gai-chu-xian-de-mu-lu"></a>

#### src <a href="#src" id="src"></a>

有一些 Go 项目确实包含 `src` 文件夹，但通常只有在开发者是从 Java（这是 Java 中一个通用的模式）转过来的情况下才会有。如果可以的话请不要使用这种 Java 模式。你肯定不希望你的 Go 代码和项目看起来向 Java。

不要将项目级别的 `/src` 目录与 Go 用于其工作空间的 `/src` 目录混淆，就像 [How to Write Go Code](https://golang.org/doc/code.html)中描述的那样。`$GOPATH`环境变量指向当前的工作空间（默认情况下指向非 Windows 系统中的$HOME/go）。此工作空间包括顶级 `/pkg`，`/bin` 和 `/src` 目录。实际的项目最终变成 `/src` 下的子目录，因此，如果项目中有 `/src` 目录，则项目路径将会变成：`/some/path/to/workspace/src/your_project/src/your_code.go`。请注意，使用 Go 1.11，可以将项目放在 `GOPATH` 之外，但这并不意味着使用此布局模式是个好主意。

### 其他文件 <a href="#qi-ta-wen-jian" id="qi-ta-wen-jian"></a>

#### Makefile <a href="#makefile" id="makefile"></a>

在任何一个项目中都会存在一些需要运行的脚本，这些脚本文件应该被放到 `/scripts` 目录中并**由 Makefile 触发**。

### 小结 <a href="#xiao-jie" id="xiao-jie"></a>

每个公司、组织内部都有自己的组织方式，但每个项目都应该有一定的规范。虽然这种规范的约定没有那么强制，但是只要达成了一致之后，对于团队中组员快速理解和入门项目都是很有帮助的。有时候**一些规范，就是团队的共同语言，定好了规范，减少了不必要的重复沟通，有利于提高整体的效率**。

项目目录也一样，本篇文章讲的是参考 [golang-standards](https://github.com/golang-standards/project-layout) 提供的规范。但是，**最重要的还是要与自己的团队商量，讨论并整理出适合自己的一套项目目录规范**。

**一致的项目目录规范，有助于组员快速理解其他人的代码，不容易造成团队的”单点故障“；团队团结一致，共同维护和升级项目目录结构，可不断沉淀，不断提高效率，减少犯错**。

### 延伸阅读 <a href="#yan-shen-yue-du" id="yan-shen-yue-du"></a>

* [在 Golang 上使用整洁架构（Clean Architecture）](https://makeoptim.com/golang/clean-architecture)
* [在 Golang 上使用整洁架构（Clean Architecture）-2](https://makeoptim.com/golang/clean-architecture-2)
* [在 Golang 上使用整洁架构（Clean Architecture）-3](https://makeoptim.com/golang/clean-architecture-3)
* [Effective Go 中文](https://makeoptim.com/golang/effective-go)
* [Code Review 规范](https://makeoptim.com/golang/standards/code-review-comments)
* [golang 1.14 1.15 1.16 新特性一览](https://makeoptim.com/golang/new-features)
* [golang 1.18 泛型、模糊测试、工作区、性能提升，里程碑式的版本](https://makeoptim.com/golang/1-18)
* [golang 1.18 泛型教程](https://makeoptim.com/golang/generics-tutorial)
* [golang 1.19 工具、运行时、库、性能，改良版](https://makeoptim.com/golang/1-19)

### 参考 <a href="#can-kao" id="can-kao"></a>

* [https://github.com/golang-standards/project-layout](https://github.com/golang-standards/project-layout)
