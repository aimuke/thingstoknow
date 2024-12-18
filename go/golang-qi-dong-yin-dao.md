# Golang 启动引导

## 介绍

Go 程序的一生有编码、编译、汇编、链接、运行、退出等步骤。它的每一步具体如何进行呢？本文就来讲解 Golang 程序启动引导过程。

Go 程序启动后需要对自身运行时进行初始化，其真正的程序入口由 runtime 包控制。以 AMD64 架构上的 Linux 和 macOS 为例，分别位于：`src/runtime/rt0_linux_amd64.s` 和 `src/runtime/rt0_darwin_amd64.s`。在 plan9 中 TEXT 是一个指令，用来定义一个函数。

```go
TEXT _rt0_amd64_linux(SB),NOSPLIT,$-8
	JMP	_rt0_amd64(SB)
TEXT _rt0_amd64_darwin(SB),NOSPLIT,$-8
	JMP	_rt0_amd64(SB)
```

可见，两者均跳转到了 `_rt0_amd64` 函数。这种做法符合直觉，在程序编译为机器码之后，依赖特定 CPU 架构的指令集，而操作系统的差异则是直接反应在运行时进行不同的系统级操作上，例如：系统调用。

rt0 其实是 `runtime0` 的缩写，意为运行时的创生，随后所有创建的都是 1 为后缀。

## 入口参数

操作系统通过入口参数的约定与应用程序进行沟通，为了支持从系统给运行时传递参数，Go 程序在进行引导时将对这部分参数进行处理。程序刚刚启动时，栈指针 SP 的前两个值分别对应 argc 和 argv，分别存储参数的数量和具体的参数的值：

```go
TEXT _rt0_amd64(SB),NOSPLIT,$-8
	MOVQ	0(SP), DI	// argc
	LEAQ	8(SP), SI	// argv
	JMP	runtime·rt0_go(SB)

TEXT runtime·rt0_go(SB),NOSPLIT,$0
	// 将参数向前复制到一个偶数栈上
	MOVQ	DI, AX			// argc
	MOVQ	SI, BX			// argv
	SUBQ	$(4*8+7), SP	// 2args 2auto
	ANDQ	$~15, SP
	MOVQ	AX, 16(SP)
	MOVQ	BX, 24(SP)

	// 初始化 g0 执行栈
	MOVQ	$runtime·g0(SB), DI			// DI = g0
	LEAQ	(-64*1024+104)(SP), BX
	MOVQ	BX, g_stackguard0(DI)		// g0.stackguard0 = SP + (-64*1024+104)
	MOVQ	BX, g_stackguard1(DI)		// g0.stackguard1 = SP + (-64*1024+104)
	MOVQ	BX, (g_stack+stack_lo)(DI)	// g0.stack.lo    = SP + (-64*1024+104)
	MOVQ	SP, (g_stack+stack_hi)(DI)	// g0.stack.hi    = SP

	// 确定 CPU 处理器的信息
	MOVL	$0, AX
	CPUID			// CPUID 会设置 AX 的值
	MOVL	AX, SI
	(...)
```

## 线程本地存储 TLS

确定完程序入口参数和 CPU 处理器信息之后，一个影响运行时非常重要的操作便是本地线程存储 （Thread Local Storage, TLS）。

```go
TEXT runtime·rt0_go(SB),NOSPLIT,$0
	(...)
#ifdef GOOS_darwin
	// skip TLS setup on Darwin
	JMP ok
#endif

	LEAQ	runtime·m0+m_tls(SB), DI  // DI = m0.tls
	CALL	runtime·settls(SB)        // 将 TLS 地址设置到 DI

	// 使用它进行存储，确保能正常运行
	get_tls(BX)
	MOVQ	$0x123, g(BX)
	MOVQ	runtime·m0+m_tls(SB), AX
	CMPQ  AX, $0x123        // 判断 TLS 是否设置成功
	JEQ 2(PC)               // 如果相等则向后跳转两条指令
	CALL  runtime·abort(SB) // 使用 INT 指令执行中断
ok:
	// 程序刚刚启动，此时位于主线程
	// 当前栈与资源保存在 g0
	// 该线程保存在 m0
	MOVQ	TLS, BX
	LEAQ	runtime·g0(SB), CX
	MOVQ	CX, g(BX)
	LEAQ	runtime·m0(SB), AX

	MOVQ	CX, m_g0(AX) // m->g0 = g0
	MOVQ	AX, g_m(CX)  // g0->m = m0
	(...)
```

而 `g0` 和 `m0` 是一组全局变量，在程序运行之初就已经存在。除了程序参数外，会首先将 `m0` 与 `g0` 通过指针互相关联。

```go
TEXT runtime·settls(SB),NOSPLIT,$32
	ADDQ	$8, DI	// DI = DI + 8, ELF 格式使用 -8(FS)
	MOVQ	DI, SI					// SI = DI
	MOVQ	$0x1002, DI				// 0x1002 == ARCH_SET_FS
	MOVQ	$SYS_arch_prctl, AX
	SYSCALL
	CMPQ	AX, $0xfffffffffffff001	// 验证是否成功
	JLS	2(PC)					
	MOVL	$0xf1, 0xf1				// 崩溃
	RET
```

可以看到到此函数进行 `arch_prctl` 系统调用并 `ARCH_SET_FS` 作为参数传递，为 `FS` 段寄存器设置了基础。

## 早期校验与系统级初始化

在正式初始化运行时组件之前，还需要做一些校验和系统级的初始化工作，这包括：运行时类型检查，系统参数的获取以及影响内存管理和程序调度的相关常量的初始化。

```go
TEXT runtime·rt0_go(SB),NOSPLIT,$0
	(...)
	CALL	runtime·check(SB)

	MOVL	16(SP), AX		// 复制 argc
	MOVL	AX, 0(SP)
	MOVQ	24(SP), AX		// 复制 argv
	MOVQ	AX, 8(SP)
	CALL	runtime·args(SB)
	CALL	runtime·osinit(SB)
	(...)
```

#### 运行时类型检查 <a href="#yun-hang-shi-lei-xing-jian-cha" id="yun-hang-shi-lei-xing-jian-cha"></a>

```go
// runtime/runtime1.go
func check() {
	var (
		a     int8
		b     uint8
		(...)
	)
	(...)

	// 校验 int8 类型 sizeof 是否为 1，下同
	if unsafe.Sizeof(a) != 1 { throw("bad a") }
	if unsafe.Sizeof(b) != 1 { throw("bad b") }
	(...)
}
```

#### 系统参数、处理器与内存常量 <a href="#xi-tong-can-shu-chu-li-qi-yu-nei-cun-chang-liang" id="xi-tong-can-shu-chu-li-qi-yu-nei-cun-chang-liang"></a>

argc, argv 作为来自操作系统的参数传递给 args 处理程序参数的相关事宜。

```go
// runtime/runtime1.go
func args(c int32, v **byte) {
	argc = c
	argv = v
	sysargs(c, v)

```



![](<../.gitbook/assets/image (104).png>)

args 函数将参数指针保存到了 argc 和 argv 这两个全局变量中，供其他初始化函数使用，而后调用了平台特定的 sysargs。对于 Darwin 系统而言，只负责获取程序的 executable\_path：

这个参数用于设置 os 包中的 executablePath 变量。

而在 Linux 平台中，这个过程就变得复杂起来了。与 Darwin 使用 mach-o 不同，Linux 使用 ELF 格式 \[Matz et al. 2014]。ELF 除了 argc, argv, envp 之外，会携带辅助向量（auxiliary vector）将某些内核级的信息传递给用户进程，例如内存物理页大小。具体结构如图 5.1 所示。

对照图 5.1 的词表，我们能够很容易的看明白 sysargs 在 Linux amd64 下作的事情：

```go
// runtime/os_linux.go

// physPageSize 是操作系统的内存物理页字节大小。
// 内存页的映射和反映射操作必须以 physPageSize 的整数倍完成
var physPageSize uintptr

func sysargs(argc int32, argv **byte) {
	// 跳过 argv, envp 来获取 auxv
	n := argc + 1
	for argv_index(argv, n) != nil { n++ }

	n++ // 跳过 NULL 分隔符

	// 尝试读取 auxv
	auxv := (*[1 << 28]uintptr)(add(unsafe.Pointer(argv), uintptr(n)*sys.PtrSize))
	if sysauxv(auxv[:]) != 0 {
		return
	}

	// 处理无法读取 auxv 的情况：
	// 一种方法是尝试读取 /proc/self/auxv。
	// 如果这个文件不存在，还可以尝试调用 mmap 等内存分配的系统调用直接测试物理页的大小。
	(...)
}

func sysauxv(auxv []uintptr) int {
	var i int
	// 依次读取 auxv 键值对
	for ; auxv[i] != _AT_NULL; i += 2 {
		tag, val := auxv[i], auxv[i+1]
		switch tag {
		case _AT_PAGESZ:
			// 读取内存页的大小
			physPageSize = val
			// 这里其实也可能出现无法读取到物理页大小的情况，但后续再内存分配器初始化的时候还会对
			// physPageSize 的大小进行检查，如果读取失败则无法运行程序，从而抛出运行时错误
		(...)
		}
		(...)
	}
	return i / 2
}
```

因此对于 Linux 而言，物理页大小在 sysargs 中便能直接完成初始化。

最后是，osinit 完成对 CPU 核心数的获取，因为这与调度器有关。而 Darwin 上由于使用的是 mach-o 格式，在此前的 sysargs 上 还没有确定内存页的大小，因而在这个函数中，还会额外使用 sysctl 完成物理页大小的查询。

```go
var ncpu int32

// Linux
func osinit() {
	ncpu = getproccount()
}

// Darwin
func osinit() {
	ncpu = getncpu()
	physPageSize = getPageSize() // 内部使用 sysctl 来获取物理页大小.
}
```



Darwin 从操作系统发展来看，是从 NeXTSTEP 和 FreeBSD 2.x 发展而来的后代，macOS 系统调用的特殊之处在于它提供了两套调用接口，一个是 Mach 调用，另一个则是 POSIX 调用。 Mach 是 NeXTSTEP 遗留下来的产物，其 BSD 层本质上是对 Mach 内核的一层封装。 尽管用户态进程可以直接访问 Mach 调用，但出于通用性的考虑，物理页大小获取的方式是通过 POSIX sysctl 这个系统调用进行获取 \[Bacon, 2007]。

事实上 Linux 与 Darwin 下的系统调用如何参与到 Go 程序中去稍有不同，我们暂时不做深入讨论，留到以后再统一分析。

可以看出，对运行时最为重要的两个系统级参数：CPU 核心数与内存物理页大小。

#### 运行时组件核心 <a href="#yun-hang-shi-zu-jian-he-xin" id="yun-hang-shi-zu-jian-he-xin"></a>

万事俱备只欠东风，对于 Go 运行时而言，最后的这三个函数及其后续调用关系完整实现了整个程序的全部运行时机制的准备工作：

```go
TEXT runtime·rt0_go(SB),NOSPLIT,$0
	(...)
	// 调度器初始化
	CALL	runtime·schedinit(SB)

	// 创建一个新的 goroutine 来启动程序
	MOVQ	$runtime·mainPC(SB), AX
	PUSHQ	AX
	PUSHQ	$0			// 参数大小
	CALL	runtime·newproc(SB)
	POPQ	AX
	POPQ	AX

	// 启动这个 M，mstart 应该永不返回
	CALL	runtime·mstart(SB)
	(...)
	RET
```

其中：

1. schedinit：进行各种运行时组件初始化工作，这包括我们的调度器与内存分配器、回收器的初始化
2. newproc：负责根据主 goroutine（即 main）入口地址创建可被运行时调度的执行单元
3. mstart：开始启动调度器的调度循环

编译器负责生成了 main 函数的入口地址，runtime.mainPC 在数据段中被定义为 runtime.main 保存主 goroutine 入口地址：

```go
DATA	runtime·mainPC+0(SB)/8,$runtime·main(SB)
GLOBL	runtime·mainPC(SB),RODATA,$8
```

最后我们来大致浏览一下 schedinit 的全貌。schedinit 函数名表面上是调度器的初始化，但实际上它包含了所有核心组件的初始化工作。

```go
// src/runtime/proc.go
func schedinit() {
	_g_ := getg()
	(...)

	// 栈、内存分配器、调度器相关初始化
	sched.maxmcount = 10000	// 限制最大系统线程数量
	stackinit()			// 初始化执行栈
	mallocinit()		// 初始化内存分配器
	mcommoninit(_g_.m)	// 初始化当前系统线程
	(...)

	gcinit()	// 垃圾回收器初始化
	(...)

	// 创建 P
	// 通过 CPU 核心数和 GOMAXPROCS 环境变量确定 P 的数量
	procs := ncpu
	if n, ok := atoi32(gogetenv("GOMAXPROCS")); ok && n > 0 {
		procs = n
	}
	procresize(procs)
	(...)
}
```

我们最感兴趣的三大运行时组件在如下函数签名中进行大量初始化工作：

* stackinit() goroutine 执行栈初始化
* mallocinit() 内存分配器初始化
* mcommoninit() 系统线程的部分初始化工作
* gcinit() 垃圾回收器初始化
* procresize() 根据 CPU 核心数，初始化系统线程的本地缓存

## 小结

我们通过一个简化的调用关系图来对本节中我们观察到的程序启动流程。

![](<../.gitbook/assets/image (120).png>)

据分析我们可以看到，Go 程序既不是从 `main.main` 直接启动，也不是从 `runtime.main` 直接启动。相反，其实际的入口位于 `runtime._rt0_amd64_*`。随后会转到 `runtime.rt0_go` 调用。在这个调用中，除了进行运行时类型检查外，还确定了两个很重要的运行时常量，即处理器核心数以及内存物理页大小。

程序引导和初始化工作是整个运行时最关键的基础步骤之一。在 `schedinit` 这个函数的调用过程中，还会完成整个程序运行时的初始化，包括调度器、执行栈、内存分配器、调度器、垃圾回收器等组件的初始化。最后通过 `newproc` 和 `mstart` 调用进而开始由调度器转为执行主 `goroutine`。

运行时组件的内容我们留到组件各自的章节中进行讨论，我们在下一节中着先着重讨论当一切都初始化好后，程序的正式启动过程，即 `runtime.main`。

## References

* [原文 Golang 启动引导](https://loulan.me/post/golang-boot/) ,楼兰
