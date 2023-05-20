# 我来告诉你Go项目标准结构如何布局

[Tony Bai](https://tonybai.com/)

![](https://tonybai.com/wp-content/uploads/the-standard-layout-of-go-project-1.png)

[本文永久链接](https://tonybai.com/2022/04/28/the-standard-layout-of-go-project) – https://tonybai.com/2022/04/28/the-standard-layout-of-go-project

每当我们编写一个非hello world的实用Go程序或库时，我们都会在项目结构、代码风格以及标识符命名这三个“门槛”前面踯躅徘徊许久，甚至始终得不到满意答案。

本文将通过《Go语言精进之路：从新手到高手的编程思想、方法与技巧》这本书的内容来详细看一看Go项目结构这道“门槛”应该如何迈过，以帮助大家能更快地深入Go语言的核心腹地，提高学习效率。

***

除非是像hello world这样的简单程序，但凡我们编写一些实用程序或库，我们都会遇到采用什么样的项目结构（project structure）的问题（通常一个Go项目对应一个仓库repository）。在Go语言中，项目结构的同样十分重要，因为这决定了项目内部包的布局以及包依赖关系是否合理，同时还会影响到外部项目对该项目中包的依赖与引用。

#### 1. Go项目的项目结构

我们先来看看世界上第一个Go项目- Go语言自身的项目结构是什么样的。

Go项目的项目结构自1.0版本发布以来一直十分稳定，直到现在Go项目的顶层结构基本没有大的改变。截至go项目commit 1e3ffb0c（2019.5.14），go项目结构如下：

```
$ tree -LF 1 ~/go/src/github.com/golang/go
./go
├── api/
├── AUTHORS
├── CONTRIBUTING.md
├── CONTRIBUTORS
├── doc/
├── favicon.ico
├── lib/
├── LICENSE
├── misc/
├── PATENTS
├── README.md
├── robots.txt
├── src/
└── test/
```

作为Go语言的“创世项目”，其项目结构的布局对后续的其他Go语言项目具有重要的参考意义，尤其是早期Go项目中src目录下面的结构，更是在后续被Go社区作为Go应用项目结构的模板被广泛使用。我们以早期的Go 1.3版本的src目录下的结构为例：

```
$ tree -LF 1 ./src
./src
├── all.bash*
├── all.bat
├── all.rc*
├── clean.bash*
├── clean.bat
├── clean.rc*
├── cmd/
├── lib9/
├── libbio/
├── liblink/
├── make.bash*
├── make.bat
├── Make.dist
├── make.rc*
├── nacltest.bash*
├── pkg/
├── race.bash*
├── race.bat
├── run.bash*
├── run.bat
├── run.rc*
└── sudo.bash*
```

关于上面src目录下面的结构，我总结了三个特点：

1）代码构建的脚本源文件放在src下面的顶层目录下；\
2）src下的二级目录cmd下面存放着go工具链相关的可执行文件（比如：go、gofmt等）的主目录以及它们的main包源文件；

```
$ tree -LF 1 ./cmd
./cmd
...
├── 6a/
├── 6c/
├── 6g/
...
├── cc/
├── cgo/
├── dist/
├── fix/
├── gc/
├── go/
├── gofmt/
├── ld/
├── nm/
├── objdump/
├── pack/
└── yacc/
```

3）src下的二级目录pkg下面存放着上面cmd下各工具链程序依赖的包、go运行时以及go标准库的源文件

```
$ tree -LF 1 ./pkg
./pkg
...
├── flag/
├── fmt/
├── go/
├── io/
├── log/
├── math/
...
├── syscall/
├── testing/
├── text/
├── time/
├── unicode/
└── unsafe/
```

在Go 1.3版本以后至今，Go项目下的src目录中发生了几次结构上的变动：

* Go 1.4版本中删除了Go源码树中src/pkg/xxx中pkg这一层级目录而直接使用src/xxx；
* Go 1.4版本在src下面增加internal目录，用于存放无法被外部导入仅Go项目自用的包；
* Go 1.6版本在src下面增加vendor目录，但Go项目自身真正启用vendor机制是在Go 1.7版本中。vendor目录中存放了go项目自身对外部项目的依赖，主要是golang.org/x下的各个包，包括：net、text、crypto等。该目录下的包会在每次Go版本发布时做更新；
* Go 1.13版本在src下面增加了go.mod和go.num，实现了go项目自身的go module迁移，go项目内所有包被放入名为std的module下面，其依赖的包依然是golang.org/x下的各个包

```
// Go 1.13版本go项目src下面的go.mod
module std

go 1.12

require (
    golang.org/x/crypto v0.0.0-20200124225646-8b5121be2f68
    golang.org/x/net v0.0.0-20190813141303-74dc4d7220e7
    golang.org/x/sys v0.0.0-20190529130038-5219a1e1c5f8 // indirect
    golang.org/x/text v0.3.2 // indirect
)
```

下面是最新的Go 1.16版本src目录下的完整布局：

```
├── Make.dist
├── README.vendor
├── all.bash*
├── all.bat
├── all.rc*
├── bootstrap.bash*
├── buildall.bash*
├── clean.bash*
├── clean.bat
├── clean.rc*
├── cmd/
├── cmp.bash
├── go.mod
├── go.sum
├── internal/
├── make.bash*
├── make.bat
├── make.rc*
├── race.bash*
├── race.bat
├── run.bash*
├── run.bat
├── run.rc*
├── testdata/
...
└── vendor/
```

#### 2. Go语言典型项目结构

**（1）Go项目结构的最小标准布局**

关于Go应用项目结构的标准布局是什么样子的，Go官方团队始终没有给出参考标准。不过作为Go语言项目的技术负责人，Russ Cox在一个开源项目的issue中给出了他关于Go项目结构的最小标准布局[Russ Cox关于Go项目结构的最小标准布局的想法](https://github.com/golang-standards/project-layout/issues/117#issuecomment-828503689)的想法。他认为的Go项目的最小标准布局应该是如下这样的：

```
// 在Go项目仓库根路径下

- go.mod
- LICENSE
- xx.go
- yy.go
...

或

- go.mod
- LICENSE
- package1
  - package1.go
- package2
  - package2.go
...
```

至于pkg、cmd、docs这些目录不应该成为Go项目标准结构的一部分，至少不是必须的。笔者认为Russ Cox给出的最小标准布局与Go一贯崇尚的“简单”哲学是一脉相承的，这个布局很灵活，可以满足各种Go项目的需求。

但是在Russ Cox阐述上述最小标准之前，Go社区其实是处于“无标准”状态的，早期Go语言自身项目的结构布局对现存的大量Go开源项目的影响依然持久，对于一些规模稍大些的Go应用项目，我们势必会在上述的“最小标准布局”的基础上做出扩展。而这种扩展也显然也不会是盲目的，而还是会参考Go语言项目自身的结构布局，于是我们就有了下面的非官方标准的建议结构布局。

**（2）以构建二进制可执行文件为目的的Go项目结构**

基于Go语言项目自身的早期结构以及后续演进，Go社区在多年的Go语言实践积累后逐渐形成了一种典型项目结构，这种结构与Russ Cox的最小标准布局是兼容的，如图1所示。

![](https://tonybai.com/wp-content/uploads/the-standard-layout-of-go-project-2.jpg)\


图1 Go语言典型项目结构（以构建二进制可执行文件为目的的Go项目）

上面就是一个支持构建二进制可执行文件（在cmd下）的典型Go项目的结构，我们分别来看一下各个重要目录的用途：

* cmd目录：存放项目要编译构建的可执行文件对应的main包的源文件。如果有多个可执行文件需要构建，每个可执行文件的main包单独放在一个子目录中，比如图中的app1、app2；cmd目录下的各app的main包将整个项目的依赖连接在一起；并且通常来说，main包应该很简洁。我们在main包中会做一些命令行参数解析、资源初始化、日志设施初始化、数据库连接初始化等工作，之后就会将程序的执行权限交给更高级的执行控制对象；也有一些go项目将cmd这个名字改为app，但其功用并没有变；
* pkg目录：存放项目自身要使用、同样也是可执行文件对应main包所要依赖的库文件；同时该目录下的包还可以被外部项目引用，算是项目导出包的一个“聚合”；也有些项目将pkg这个名字改为lib，但目录用途不变；由于Go语言项目自身在1.4版本中去掉了pkg这一层目录，因此也有一些项目直接将包平铺到项目根路径下，但笔者认为对于一些规模稍大的项目，过多的包会让项目顶层目录不再简洁，显得很“拥挤”，因此我个人建议对于复杂的Go项目，保留pkg目录未尝不可。
* Makefile：这里的Makefile是项目构建工具所用脚本的“代表”，它可以代表任何第三方构建工具所用的脚本。Go并没有内置如make、bazel等级别的项目构建工具，对于一些规模稍大的项目而言，项目构建工具似乎还不可缺少。在Go典型项目中，项目构建工具的脚本一般放在项目顶层目录下，比如这里的Makefile；对于构建脚本较多的项目，也可以建立build目录，并将构建脚本的规则属性文件、子构建脚本放入其中。
* go.mod和go.sum：Go语言包依赖管理使用的配置文件。Go 1.11版本引入go modules机制，Go 1.16版本中，go module成为默认的依赖包管理和构建机制。因此对于新的Go项目，我们建议基于go modules进行包依赖管理。对于没有使用go modules进行包管理的项目（可能主要是一些使用go 1.11以前版本的go项目），这里可以换为dep的Gopkg.toml和Gopkg.lock或者glide的glide.yaml和glide.lock等。
* vendor目录（可选）：vendor是Go 1.5版本引入的用于在项目本地缓存特定版本依赖包的机制，在go modules机制引入前，基于vendor可以实现可重现的构建（reproducible build），保证基于同一源码构建出的可执行程序是等价的。go modules本身就可以实现可重现的构建，而无需vendor，当然go module机制也保留了vendor目录（通过go mod vendor可以生成vendor下的依赖包；通过go build -mod=vendor可以实现基于vendor的构建），因此这里将vendor目录视为一个可选目录。一般我们仅保留项目根目录下的vendor目录，否则会造成不必要的依赖选择的复杂性。

Go 1.11引入的module是一组同属于一个版本管理单元的包的集合。并且Go支持在一个项目/仓库中存在多个module，但这种管理方式可能要比一定比例的代码重复引入更多的复杂性。因此，如果项目结构中存在版本管理的“分歧”，比如：app1和app2的发布版本并不总是同步的，那么笔者建议将项目拆分为多个项目（仓库），每个项目单独作为一个module进行单独的版本管理和演进。

**（3）以只构建库为目的的Go项目结构**

Go 1.4发布时，Go语言项目自身去掉了src下的pkg这一层目录，这个结构上的改变对那些以只构建库为目的的Go库类型项目结构有着一定的影响。我们来看一个典型的Go语言库类型项目的结构布局，见图2。

![](https://tonybai.com/wp-content/uploads/the-standard-layout-of-go-project-3.jpg)\


图2 Go语言库项目结构

我们看到库类型项目结构与Go项目的最小标准布局也是兼容的，但相比于以构建二进制可执行文件为目的的Go项目要简单一些：

* 去除了cmd和pkg两个子目录：由于仅构件库，没必要保留存放二进制文件main包源文件的cmd目录；由于Go库项目的初衷一般都是为了对外部（开源或组织内部公开）暴露API，因此也没有必要将其单独聚合到pkg目录下面了；
* vendor也不再是可选目录：对于库类型项目而言，我们不推荐在项目中放置vendor目录去缓存库自身的第三方依赖，库项目仅通过go.mod（或其他包依赖管理工具的manifest文件）明确表述出该项目依赖的模块或包以及版本要求即可。

**（4）关于internal目录**

无论是上面哪种类型的Go项目，对于不想暴露给外部引用，仅限项目内部使用的包，在项目结构上可以通过Go 1.4版本中引入的internal包机制来实现。以库项目为例，最简单的方式就是在顶层加入一个internal目录，将不想暴露到外部的包都放在该目录下，比如下面项目结构中的ilib1、ilib2：

```
// 带internal的Go库项目结构
$tree -F ./chapter2/sources/GoLibProj
GoLibProj
├── LICENSE
├── Makefile
├── README.md
├── go.mod
├── internal/
│   ├── ilib1/
│   └── ilib2/
├── lib.go
├── lib1/
│   └── lib1.go
└── lib2/
    └── lib2.go
```

这样，根据go internal机制的作用原理，internal目录下的ilib1、ilib2可以被以GoLibProj目录为根目录的其他目录下的代码(比如lib.go、lib1/lib1.go等）所导入和使用，但是却不可以为GoLibProj目录以外的代码所使用，从而实现选择性的暴露API包。当然internal也可以放在项目结构中的任一目录层级中，关键是项目结构设计人员明确哪些要暴露到外层代码，哪些仅用于同级目录或子目录中。

对于以构建二进制可执行文件类型为目的的项目来说，我们同样可以将不想暴露给外面的包聚合到项目顶层路径下的internal下，与暴露给外部的包的聚合目录pkg遥相呼应。

#### 3. 小结

本文详细介绍了Go语言项目的结构布局的来龙去脉以及Go项目结构的事实标准。文中的两个针对构建二进制可执行文件类型以及库类型的项目参考结构是Go社区在多年实践后得到公认且使用较为广泛的项目结构，并且它们与Russ Cox提出的Go项目最小标准布局是兼容的，对于稍大型的Go项目来说很具有参考价值。但它们也不是必须的，在Go语言早期，很多项目将所有源文件都放在位于项目根目录下的根包中的做法在一些小规模项目中同样工作得很好。

对于以构建二进制可执行文件类型为目的的项目来说，受Go 1.4项目结构影响，将pkg这一层次目录去掉也是很多项目选择的结构布局方式。

上述的参考项目结构与产品设计开发领域的“最小可行产品”（minimum viable product，简称为mvp）的思路有些异曲同工，开发者可以在这样一个最小的“项目结构核心”的基础上根据实际需要对其进行扩展。

如果您想要了解更多有关Go项目结构与布局的内容，推荐您详细阅读我的新作《Go语言精进之路：从新手到高手的编程思想、方法与技巧》。



## 原文地址 [https://tonybai.com/2022/04/28/the-standard-layout-of-go-project/](https://tonybai.com/2022/04/28/the-standard-layout-of-go-project/)
