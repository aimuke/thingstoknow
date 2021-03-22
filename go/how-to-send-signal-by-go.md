# 如何在golang中发送中断信号？

标签： go signals interrupt interrupt-handling

我目前正在尝试实现一个可以在Google Go中调用中断信号的函数。我知道如何使用`signal.Notify(interruptChannel, os.Interrupt)`拦截来自控制台的中断信号，但是，我找不到实际发送中断信号的方法。我发现 [can send a signal to a process](https://golang.org/pkg/os/#Process.Signal)，但我不确定这是否可用于发送顶级中断信号。

有没有办法从golang函数中发送一个中断信号，该函数可以被正在侦听系统中断信号的任何东西捕获，或者是golang不支持的东西？

## 使用syscall.Kill

假设你正在使用这样的东西捕获中断信号

```go
var stopChan = make(chan os.Signal, 2)
signal.Notify(stopChan, os.Interrupt, syscall.SIGTERM, syscall.SIGINT)

<-stopChan // wait for SIGINT
```

在代码中的任何位置使用以下命令将中断信号发送到上述等待部分。

```go
syscall.Kill(syscall.Getpid(), syscall.SIGINT)
```

或者，如果您位于定义了stopChan变量的同一个包中。从而使它可以访问。你可以这样做。

```go
stopChan <- syscall.SIGINT
```

或者您可以将stopChan定义为全局变量（使大写字母中的第一个字母实现相同），然后您也可以从其他包中发送中断信号。

```go
Stopchan <- syscall.SIGINT
```

## 使用process

使用[FindProcess](https://godoc.org/os#FindProcess)，[StartProcess](https://godoc.org/os#StartProcess)或其他方式获取流程。调用[Signal](https://godoc.org/os#Process.Signal)发送中断：

```text
 err := p.Signal(os.Interrupt)
```

这会将信号发送到目标进程（假设调用进程有权这样做）并调用目标进程可能对SIGINT有的任何信号处理程序。



## Reference

* [如何在golang中发送中断信号？](https://www.thinbug.com/q/40498371)

