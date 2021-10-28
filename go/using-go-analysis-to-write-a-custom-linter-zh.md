# \[译] 使用 go/analysis 包实现自定义的 linter

**独家号 一亩三分地 作者 @colobu原文链接**

Fatih Arslan 是 [vim-go](https://github.com/fatih/vim-go)、[gomodifytags](https://github.com/fatih/gomodifytags)等开源项目的作者，DigitalOcean的一名软件工程师。这是一篇关于使用go/analysis写自定义的linter工具的介绍，原文: [Using go/analysis to write a custom linter](https://arslan.io/2019/06/13/using-go-analysis-to-write-a-custom-linter/)

如果你问人们为什么爱Go爱的那么深沉，那么答案之一就是工具。原因在于使用Go写工具真是一件容易的事，尤其是为Go语言写一些定制的工具。lint工具就是其中的工具之一。如果你已经使用了Go，那么你一定已经知道和使用了其中的一些工具，比如go vet、goline、staticcheck等等。

所有这些工具都是使用go/{ast, packages, types, ...}等包去解析(parse)和解释(interpret) Go代码，但是，并没有一个通用的框架更容易和有效地分析Go代码。如果你使用上面的包，你就不得不实现很多让你讨厌的东西(flag解析，AST遍历，上下文信息的传递等等)。

为了改善现状，为今后的工作奠定更好的基础，Go作者们引入了一个新的软件包：[go/analysis](https://godoc.org/golang.org/x/tools/go/analysis)。

go/analysis提供了一个实现checker的通用接口。checker就是能够报告错误的分析器。go/analysis包仍处于开发之中，接口和定义随时有变化，因此请确保及时更新。

在这篇文章中，我们将使用新的go/analysis包编写一个定制的linter（也叫做checker）。如果您还没有使用过一些这类的工具来分析和检查go源代码（例如go/parser和go/ast包），请先阅读我以前的博客文章：[编写go工具的终极指南](https://arslan.io/2017/09/14/the-ultimate-guide-to-writing-a-go-tool/)，这是理解本文后续部分所必需的。

现在让我们开始写我们自己的linter工具吧。

## 自定义的linter的需求

首先让我们定义我们这个linter的需求(功能)。它非常简单，我们将这个linter工具称之为addlint。这个工具的功能就是报告整数相加的使用情况：

```
3 + 2
```

举例来说，假定我们有下面一个简单的main包:

```go
package main
import "fmt"
func main() {
	sum := 3 + 2
	fmt.Printf("Sum: %d\n", sum)
}
```

如果我们运行addlint检查这个文件，它应该报告如下的信息:

```
$ addlint foo.go
/Users/fatih/foo.go:6:9: integer addition found: '3 + 2'
```

它也可以用来分析包，就像其它的的Go工具一样:

```
$ addlint github.com/my/repo
/Users/fatih/repo/foo.go:6:9: integer addition found: '3 + 2'
```

## 使用传统方式实现

在我们使用go/analysis之前，我们先用传统的底层的包比如go/parser、go/ast等实现这个checker。通过这种方式，可以帮助我们理解go/analysis所做的改进。

首先我们需要理解3 + 2是什么，Go语言中它是一个二元表达式，可以使用类型为\*ast.BinaryExpr的AST节点来表示。例如3 + 2表达式可以写做：

```go
expr := &ast.BinaryExpr{
	X: &ast.BasicLit{
		Value: "3",
		Kind:  token.INT,
	},
	Op: token.ADD,
	Y: &ast.BasicLit{
		Value: "2",
		Kind:  token.INT,
	},
}
```

使用图来表示：\


![](https://img.toutiao.io/c/cd7ca9f77f01d7ba78dab69b9b39b518)

既然现在我们知道了要查找什么节点，那么我们可以继续编写初始checker。让我们首先分析文件（我们假设cli工具只接受文件作为参数，而不是包。我们稍后将介绍如何分析包）：

```go
var files []*ast.File
fset := token.NewFileSet()
for _, goFile := range os.Args[1:] {
	f, err := parser.ParseFile(fset, goFile, nil, parser.ParseComments)
	if err != nil {
		log.Fatal(err)
	}
	files = append(files, f)
}
```

既然我们已经有了一组\[]\*ast.File，那么让我们检查它们并搜索\*ast.BinaryExpr节点。我们使用ast.Inspect()遍历AST文件：

```go
for _, file := range files {
	ast.Inspect(f, func(n ast.Node) bool {
		be, ok := n.(*ast.BinaryExpr)
		if !ok {
			return true
		}
		if be.Op != token.ADD {
			return true
		}
		if _, ok := be.X.(*ast.BasicLit); !ok {
			return true
		}
		if _, ok := be.Y.(*ast.BasicLit); !ok {
			return true
		}
		posn := fset.Position(be.Pos())
		fmt.Fprintf(os.Stderr, "%s: integer addition found: %q\n", posn, render(fset, be)
		return true
	})
}
// render returns the pretty-print of the given node
func render(fset *token.FileSet, x interface{}) string {
	var buf bytes.Buffer
	if err := printer.Fprint(&buf, fset, x); err != nil {
		panic(err)
	}
	return buf.String()
}
```

这里的主要逻辑是ast.Inspect()。我把它写得非常详细，只是为了展示所有的步骤。之后您可以在分析器中创建可重用的函数进一步简化逻辑。我们还创建了一个简单的render()函数来渲染表达式，这样我们就可以以可读的形式漂亮地打印加法。

现在，如果你对几个文件运行这个程序，你会发现它运行得很好。然而，这里仍然存在一些问题。你知道这些问题是什么吗？以下是其中一个问题：

```go
package main
import "fmt"
func main() {
	txt := "foo" + "bar"
	fmt.Printf("Txt: %s\n", txt)
}
```

如果我们对这个文件运行addlint，它将输出这个加法！但是，我们的需求是addlint应该只显示整数加法。那我们怎么解决呢？类型！\
我们需要对代码进行类型检查，以获取表达式左侧和右侧的基础类型。首先，让我们实现检查源代码：

```go
// import "go/types" and "go/importer"
conf := types.Config{Importer: importer.Default()}
// types.TypeOf() requires all three maps are populated
info := &types.Info{
	Defs:  make(map[*ast.Ident]types.Object),
	Uses:  make(map[*ast.Ident]types.Object),
	Types: make(map[ast.Expr]types.TypeAndValue),
}
_, err = conf.Check("addlint", fset, files, info)
if err != nil {
	log.Fatalln(err)
}
```

它将检查我们传入的所有文件，然后用所有必要的信息填充信息变量的映射。因为我们将使用info.TypeOf()方法，所以我们需要填充info.Defs、info.Uses和info.Types。接下来，我们将扩展ast.Inspect以检查表达式：

```go
ast.Inspect(f, func(n ast.Node) bool {
	be, ok := n.(*ast.BinaryExpr)
	if !ok {
		return true
	}
	if be.Op != token.ADD {
		return true
	}
	if _, ok := be.X.(*ast.BasicLit); !ok {
		return true
	}
	if _, ok := be.Y.(*ast.BasicLit); !ok {
		return true
	}
	isInteger := func(expr ast.Expr) bool {
		t := info.TypeOf(expr)
		if t == nil {
			return false
		}
		bt, ok := t.Underlying().(*types.Basic)
		if !ok {
			return false
		}
		if (bt.Info() & types.IsInteger) == 0 {
			return false
		}
		return true
	}
	// check that both left and right hand side are integers
	if !isInteger(be.X) || !isInteger(be.Y) {
		return true
	}
	posn := fset.Position(be.Pos())
	fmt.Fprintf(os.Stderr, "%s: integer addition found: %q\n", posn, render(fset, be)
	return true
})
```

如您所见，我们创建了一个新的isInteger()匿名函数，它检查我们传入的表达式是否为Integer类型。然后我们使用这个函数来检查\*ast.BinaryExpr的左侧和右侧。这将涵盖加号两侧不是整数的异常情况。

现在我们已经知道了如何使用底层的go{token,parser, ast,types ...}包来实现addlint程序，接下来我们使用go/analysis来改进整个cli. （注意：上面的linter仍然有许多异常的情况，为了简单起见，我将它们留作练习。如果要修复其中一些情况，请尝试检查3+2+1或a+3）

## go/analysis API

下面是我们准备使用的一个文件夹布局。这个布局非常流行，也是新的linter工具很好的起始布局:

```
.
├── addcheck
│   └── addcheck.go
├── cmd
│   └── addlint
│       └── main.go # imports addcheck
├── go.mod
└── go.sum
```

核心逻辑在addcheck包中，由main包cmd/addlint导入使用，编译后将提供addlint二进制可执行文件。

现在，让我们把目光返回到go/analysis包。

go/analysis包的核心是analysis.Analyzer类型。这种类型描述了一个分析函数：它的名称、文档、flag、与其他分析器的关系，当然还有它的逻辑。下面您可以看到它的定义（注意：为了清晰起见，有些字段和注释被省略了，我们稍后将进行探讨）：

```go
// An Analyzer describes an analysis function and its options.
type Analyzer struct {
	// The Name of the analyzer must be a valid Go identifier
	// as it may appear in command-line flags, URLs, and so on.
	Name string
	// Doc is the documentation for the analyzer.
	// The part before the first "\n\n" is the title
	// (no capital or period, max ~60 letters).
	Doc string
	// Run applies the analyzer to a package.
	// It returns an error if the analyzer failed.
	Run func(*Pass) (interface{}, error)
	// ... omitted fields
}
```

为了创建一个分析器，我们声明一个这种类型的变量。通常每个分析器都包含在一个单独的包中，然后由驱动程序导入该包（运行该工具的main包，在我们的示例中是cmd/addlint）。

让我们开始添加cmd/addlint的框架，为此，我们将创建一个包含analysis.Analyzer变量声明的addcheck包：

addcheck.go\


```go
// Package addcheck defines an Analyzer that reports integer additions
package addcheck
import (
	"errors"
	"golang.org/x/tools/go/analysis"
)
var Analyzer = &analysis.Analyzer{
	Name: "addlint",
	Doc:  "reports integer additions",
	Run:  run,
}
func run(pass *analysis.Pass) (interface{}, error) {
	return nil, errors.New("not implemented yet")
}
```

核心逻辑在run(...)函数中实现，目前我们还没有实现它， 它接受一个\*analysis.Pass类型的参数:

```go
type Pass struct {
	Fset       *token.FileSet // file position information
	Files      []*ast.File    // the abstract syntax tree of each file
	OtherFiles []string       // names of non-Go files of this package
	Pkg        *types.Package // type information about the package
	TypesInfo  *types.Info    // type information about the syntax trees
	TypesSizes types.Sizes    // function for computing sizes of types
	...
}
```

\*analysis.Pass是核心数据，可以为分析器的run函数提供信息。正如你看到的，它包含了分析源代码所有必需的类型，比如:

```
*token.FileSet
[]*ast.File
*types.Info
```

它还包含一些辅助函数，比如pass.Report()和pass.Reportf()方法来报告诊断信息。是时候来实现run函数了。

```go
func run(pass *analysis.Pass) (interface{}, error) {
	for _, file := range pass.Files {
		ast.Inspect(file, func(n ast.Node) bool {
			// check whether the call expression matches time.Now().Sub()
			be, ok := n.(*ast.BinaryExpr)
			if !ok {
				return true
			}
			if be.Op != token.ADD {
				return true
			}
			if _, ok := be.X.(*ast.BasicLit); !ok {
				return true
			}
			if _, ok := be.Y.(*ast.BasicLit); !ok {
				return true
			}
			isInteger := func(expr ast.Expr) bool {
				t := pass.TypesInfo.TypeOf(expr)
				if t == nil {
					return false
				}
				bt, ok := t.Underlying().(*types.Basic)
				if !ok {
					return false
				}
				if (bt.Info() & types.IsInteger) == 0 {
					return false
				}
				return true
			}
			// check that both left and right hand side are integers
			if !isInteger(be.X) || !isInteger(be.Y) {
				return true
			}
			pass.Reportf(be.Pos(), "integer addition found %q",
				render(pass.Fset, be))
			return true
		})
	}
	return nil, nil
}
```

似曾相识？还是和原来一样的逻辑。与以前的传统方法相比，这个函数的优雅之处在于我们不需要解析文件，也不需要类型检查，甚至不需要查找正确的位置。所有这些都集成到了go/analysis中。

## addlint 工具

现在让我们创建我们的cmd/addlint工具， main包。go/analysis包包含了几个方便的实用程序和辅助函数，可以非常容易地创建命令行 checker程序。下面您将看到cmd/addlint main包的内容:

```go
package main
import (
	"github.com/fatih/addlint/addcheck"
	"golang.org/x/tools/go/analysis/singlechecker"
)
func main() {
	singlechecker.Main(addcheck.Analyzer)
}
```

就是这么简单！如果现在编译并运行它，不带参数，您将看到以下输出：

```go
$ addlint: reports integer additions
Usage: addlint [-flag] [package]
Flags:  -V      print version and exit
  -all
        no effect (deprecated)
  -c int
        display offending line with this many lines of context (default -1)
  -cpuprofile string
        write CPU profile to this file
  -debug string
        debug flags, any subset of "fpstv"
  -flags
        print analyzer flags in JSON
  -json
        emit JSON output
  -memprofile string
        write memory profile to this file
  -source
        no effect (deprecated)
  -tags string
        no effect (deprecated)
  -trace string
        write trace log to this file
  -v    no effect (deprecated)
```

太棒了！singlechecker包自动为我们创建了一个cli程序，还添加了几个重要的flag（对于奇怪的标志，可以根据需要更改它们）。

如果我们使用它分析任何go文件，以下就是输出信息:

```go
$ cat foo.go
package main
import (
        "fmt"
)
func main() {
        sum := 3 + 2
        fmt.Printf("Sum: %s\n", sum)
}
$ addlint foo.go
/Users/fatih/foo.go:8:9: integer addition found "3 + 2"
```

我们成功地用go/analysis创建了第一个linter！使用go/analysis的好处是非常巨大的。正如你所看到的，这种新方法使事情变得更加容易，因为您不必手动解析文件、对文件进行类型检查甚至解析flag！它集成好了传统的功能，可以随时使用。与旧的传统风格相比，go/analysis包为我们做了以下工作：

* 它通过singlechecker包自动创建了一个带有所有重要标志的CLI程序。
* 它解析文件并创建了一个包含所有文件信息的列表，格式为\[]\*ast.File。
* 它检查了所有文件的类型信息，并为我们提供了一个方便的\*types.Info变量，其中包含语法树的类型信息
* 它为我们提供了方便的Reportf()来报告诊断

既然我们已经基本了解了go/analysis是如何工作的，那么让我们继续讨论它的实际的核心特性，以及是什么使它变得更好。

## 依赖其它分析器

go/analysis有一个内置的依赖关系图，如果您在一个CLI中运行多个不同的诊断程序，它可以提高检查器的性能。analysis.Analyzer可以依赖于q其它的analysis.Analyzer，如果运行go/analysis，它将确保首先按照各自的顺序获取和运行DAG（有向无环图）中的分析器。让我们用一个简单的例子来说明这一点。

如您所知，当我们在addlint中定义它们时，我省略了\*analysis.Analyzer中的几个字段。我省略的字段之一是analysis.Analyzer.Requires:

```go
// An Analyzer describes an analysis function and its options.
type Analyzer struct {
	// Requires is a set of analyzers that must run successfully
	// before this one on a given package. This analyzer may inspect
	// the outputs produced by each analyzer in Requires.
	// The graph over analyzers implied by Requires edges must be acyclic.
	//
	// Requires establishes a "horizontal" dependency between
	// analysis passes (different analyzers, same package).
	Requires []*Analyzer
	// ...
}
```

使用Requires字段，您可以定义你的分析器的依赖关系，go/analysis将确保以正确的顺序运行它们。go/analysis附带一些有用的分析器，您可以在编写自己的分析器时依赖它们。其中之一是go/analysis/passes/inspect包。

go/analysis/passes/inspect分析器提供了一个构建块，您可以使用它来代替ast.Inspect()或ast.Walk()来遍历语法文件。我们在addlint中使用了ast.Inspect()来遍历已解析的文件，以找到\*ast.BinaryExpr’s。但是，如果您有多个分析器，并且每个分析器都必须遍历语法树的话，则效率不是很高！

go/analysis/passes/inspect包比ast.Inspect()快得多，因为它在底层使用golang.org/x/tools/go/ast/inspector包。以下摘抄自是包文档：

```go
// ...
// During construction, the inspector does a complete traversal and
// builds a list of push/pop events and their node type. Subsequent
// method calls that request a traversal scan this list, rather than walk
// the AST, and perform type filtering using efficient bit sets.
//
// Experiments suggest the inspector's traversals are about 2.5x faster
// than ast.Inspect, but it may take around 5 traversals for this
// benefit to amortize the inspector's construction cost.
// If efficiency is the primary concern, do not use Inspector for
// one-off traversals.
package inspector
```

如果您的分析器只有一次遍历，那么您不需要使用这个包，但是，如果您要有多个分析器（例如go vet或staticcheck），那么go/analysis/passes/inspect是一个很好的选择。现在让我们把这个添加到addcheck包中。首先，我们添加Requires字段，依赖于inspect analyzer:

```go
var Analyzer = &analysis.Analyzer{
	Name:     "addlint",
	Doc:      "reports integer additions",
	Run:      run,
	Requires: []*analysis.Analyzer{inspect.Analyzer},
}
```

接下来我们修改run函数，导入inspector:

```go
func run(pass *analysis.Pass) (interface{}, error) {
	// get the inspector. This will not panic because inspect.Analyzer is part
	// of `Requires`. go/analysis will populate the `pass.ResultOf` map with
	// the prerequisite analyzers.
	inspect := pass.ResultOf[inspect.Analyzer].(*inspector.Inspector)
	// the inspector has a `filter` feature that enables type-based filtering
	// The anonymous function will be only called for the ast nodes whose type
	// matches an element in the filter
	nodeFilter := []ast.Node{
		(*ast.BinaryExpr)(nil),
	}
	// this is basically the same as ast.Inspect(), only we don't return a
	// boolean anymore as it'll visit all the nodes based on the filter.
	inspect.Preorder(nodeFilter, func(n ast.Node) {
		be := n.(*ast.BinaryExpr)
		if be.Op != token.ADD {
			return
		}
		if _, ok := be.X.(*ast.BasicLit); !ok {
			return
		}
		if _, ok := be.Y.(*ast.BasicLit); !ok {
			return
		}
		isInteger := func(expr ast.Expr) bool {
			t := pass.TypesInfo.TypeOf(expr)
			if t == nil {
				return false
			}
			bt, ok := t.Underlying().(*types.Basic)
			if !ok {
				return false
			}
			if (bt.Info() & types.IsInteger) == 0 {
				return false
			}
			return true
		}
		// check that both left and right hand side are integers
		if !isInteger(be.X) || !isInteger(be.Y) {
			return
		}
		pass.Reportf(be.Pos(), "integer addition found %q",
			render(pass.Fset, be))
	})
	return nil, nil
}
```

如果我们再次编译并运行这个程序，输出是一样的:

```go
$ cat foo.go
package main
import (
        "fmt"
)
func main() {
        sum := 3 + 2
        fmt.Printf("Sum: %s\n", sum)
}
$ addlint foo.go
/Users/fatih/foo.go:8:9: integer addition found "3 + 2"
```

## 多分析器

由于上面解释的内置依赖关系图和运行者（驱动程序），实现和运行多个分析器非常的容易。例如，如果您使用的是最新的go版本并运行go vet，那么实际上您使用的是多个分析器的go/analyis。cmd/vet命令的主要功能如下:

```go
package main
import (
	"golang.org/x/tools/go/analysis/unitchecker"
	"golang.org/x/tools/go/analysis/passes/asmdecl"
	"golang.org/x/tools/go/analysis/passes/assign"
	"golang.org/x/tools/go/analysis/passes/atomic"
	"golang.org/x/tools/go/analysis/passes/bools"
	"golang.org/x/tools/go/analysis/passes/buildtag"
	...
)
func main() {
	unitchecker.Main(
		asmdecl.Analyzer,
		assign.Analyzer,
		atomic.Analyzer,
		bools.Analyzer,
		buildtag.Analyzer,
		cgocall.Analyzer,
		composite.Analyzer,
		copylock.Analyzer,
		httpresponse.Analyzer,
		loopclosure.Analyzer,
		lostcancel.Analyzer,
		nilfunc.Analyzer,
		printf.Analyzer,
		shift.Analyzer,
		stdmethods.Analyzer,
		structtag.Analyzer,
		tests.Analyzer,
		unmarshal.Analyzer,
		unreachable.Analyzer,
		unsafeptr.Analyzer,
		unusedresult.Analyzer,
	)
}
```

这里unitchecker类似于singlecheckerrunner，但它接受多个分析器（注意：它还是以不同的方式解析包，但假设现在它并不重要）。您可以通过调用vet的help方法来查看所有注册的分析器：

```bash
$ ~ go tool vet help
vet is a tool for static analysis of Go programs.
vet examines Go source code and reports suspicious constructs,
such as Printf calls whose arguments do not align with the format
string. It uses heuristics that do not guarantee all reports are
genuine problems, but it can find errors not caught by the compilers.
Registered analyzers:
    asmdecl      report mismatches between assembly files and Go declarations
    assign       check for useless assignments
    atomic       check for common mistakes using the sync/atomic package
    bools        check for common mistakes involving boolean operators
    buildtag     check that +build tags are well-formed and correctly located
    cgocall      detect some violations of the cgo pointer passing rules
    composites   check for unkeyed composite literals
    copylocks    check for locks erroneously passed by value
    httpresponse check for mistakes using HTTP responses
    loopclosure  check references to loop variables from within nested functions
    lostcancel   check cancel func returned by context.WithCancel is called
    nilfunc      check for useless comparisons between functions and nil
    printf       check consistency of Printf format strings and arguments
    shift        check for shifts that equal or exceed the width of the integer
    stdmethods   check signature of methods of well-known interfaces
    structtag    check that struct field tags conform to reflect.StructTag.Get
    tests        check for common mistaken usages of tests and examples
    unmarshal    report passing non-pointer or non-interface values to unmarshal
    unreachable  check for unreachable code
    unsafeptr    check for invalid conversions of uintptr to unsafe.Pointer
    unusedresult check for unused results of calls to some functions
```

如果您checkout一些分析器，例如structtag，您将看到它的Requires依赖于inspect分析器。因此，go vet具有很高的性能，因为go/analysis框架提供了这种新的设计。

## 总结

我希望这篇文章能为你提供一个开始使用go/analysis的介绍。还有很多事情我还没有涉及。go/analysis非常强大，有许多特性使分析go代码变得简单和高效。例如，这些特性之一就是Fact。可以通过使用analysis.Fact接口实现。当您分析某些内容时，可以为给定的分析器生成Fact（注释），然后从另一个分析器导入这些Fact, 这允许您使用多个分析器创建非常强大和有效的组合。

这里写的所有代码都可以在[github.com/fatih/addlint](https://github.com/fatih/addlint) repo中找到。如果您对go/analysis有更多的问题，可以加入[gophers Slack](https://invite.slack.golangbridge.org) #tools频道，许多go开发人员在这里讨论go/analysis的问题。[Golang](https://toutiao.io/tags/Golang)\
