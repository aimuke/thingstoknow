# 使用 Go 语言实现优雅的服务器重启

Go被设计为一种后台语言，它通常也被用于后端程序中。服务端程序是GO语言最常见的软件产品。在这我要解决的问题是：如何干净利落地升级正在运行的服务端程序。

## 目标

* **不关闭现有连接**：例如我们不希望关掉已部署的运行中的程序。但又想不受限制地随时升级服务。
* **socket连接要随时响应用户请求**：任何时刻socket的关闭可能使用户返回'连接被拒绝'的消息，而这是不可取的。
* **新的进程要能够启动并替换掉旧的**。

## 原理

 在基于Unix的操作系统中，signal\(信号\)是与长时间运行的进程交互的常用方法.

* SIGTERM: 优雅地停止进程
* SIGHUP: 重启/重新加载进程 \(例如: nginx, sshd, apache\)

如果收到SIGHUP信号，优雅地重启进程需要以下几个步骤:

1. 服务器要拒绝新的连接请求，但要保持已有的连接。
2. 启用新版本的进程
3. 将socket“交给”新进程，新进程开始接受新连接请求
4. 旧进程处理完毕后立即停止。

## 停止接受连接请求

服务器程序的共同点：持有一个死循环来接受连接请求:

```go
for { 
    conn, err := listener.Accept() 
    // Handle connection
} 
```

跳出这个循环的最简单方式是在 `socket` 监听器上设置一个超时，当调用 `listener.SetTimeout(time.Now())` 后， `listener.Accept()` 会立即返回一个 `timeout err`，你可以捕获并处理：

```go
for {
  conn, err := listener.Accept()
  if err != nil {
    if nerr, ok := err.(net.Err); ok && nerr.Timeout() {
       fmt.Println(“Stop accepting connections”)
       return
    }
  }}
```

注意这个操作与关闭 listener 有所不同。这样进程仍在监听服务器端口，但连接请求会被操作系统的网络栈排队，等待一个进程接受它们。

## 启动新进程

Go提供了一个原始类型 `ForkExec` 来产生新进程.你可以与这个新进程共享某些消息，例如文件描述符或环境参数。

```go
execSpec := &syscall.ProcAttr{
  Env:   os.Environ(),
  Files: []uintptr{os.Stdin.Fd(), os.Stdout.Fd(), os.Stderr.Fd()},
}

err := syscall.ForkExec(os.Args[0], os.Args, execSpec)[…]
```

你会发现这个进程使用完全相同的参数 `os.Args` 启动了一个新进程。

## 发送socket到子进程并恢复它

正如你先前看到的，你可以将文件描述符传递到新进程，这需要一些UNIX魔法（一切都是文件），我们可以把 `socket` 发送到新进程中，这样新进程就能够使用它并接收及等待新的连接。

但 `fork-execed` 进程需要知道它必须从文件中得到socket而不是新建一个（有些兴许已经在使用了，因为我们还没断开已有的监听）。你可以按任何你希望的方法来，最常见的是通过环境变量或命令行标志。

```go
listenerFile, err := listener.File()
if err != nil {
  log.Fatalln("Fail to get socket file descriptor:", err)
}

listenerFd := listenerFile.Fd()

// Set a flag for the new process start process
os.Setenv("_GRACEFUL_RESTART", "true")
execSpec := &syscall.ProcAttr{
  Env:   os.Environ(),
  Files: []uintptr{os.Stdin.Fd(), os.Stdout.Fd(), os.Stderr.Fd(), listenerFd},
}

// Fork exec the new version of your serverfork, 
err := syscall.ForkExec(os.Args[0], os.Args, execSpec)
```

然后在程序的开始处：

```go
var listener *net.TCPListener

if os.Getenv("_GRACEFUL_RESTART") == "true" {
  file := os.NewFile(3, "/tmp/sock-go-graceful-restart")
  listener, err := net.FileListener(file)
  if err != nil {
    // handle
  }
  
  var bool ok
  listener, ok = listener.(*net.TCPListener)
  if !ok {
    // handle
  }
  
} else {
  listener, err = newListenerWithPort(12345)
}
```

文件描述没有被随机的选择为 `3`，这是因为 `uintptr` 的切片已经发送了 `fork`，监听获取了索引 `3`。留意  [隐式声明问题](http://www.qureet.com/blog/golang-beartrap/)。

## 最后一步，等待旧服务连接停止

 到此为止，就这样，我们已经将其传到另一个正在正确运行的进程，对于旧服务器的最后操作是等其连接关闭。由于标准库里提供了`sync.WaitGroup` 结构体，用go实现这个功能很简单。

每次接收一个连接，在 `WaitGroup` 上加 `1` ,然后，我们在它完成时将计数器减一：

```go
for { 
    conn, err := listener.Accept()
    wg.Add(1) 
    
    go func() { 
        handle(conn)
        wg.Done()
    }()
} 
```

至于等待连接的结束，你仅需要 `wg.Wait()`，因为没有新的连接，我们等待 `wg.Done()` 已经被所有正在运行的 `handler` 调用。

## Bonus: 不要无限制等待，给定限量的时间

有 `time.Timer`，实现很简单：

```go
timeout := time.NewTimer(time.Minute)
wait := make(chan struct{})

go func() {
  wg.Wait()
  wait <- struct{}{}
}()

select {
  case <-timeout.C:
    return WaitTimeoutError
  case <-wait:
    return nil
}
```

## 完整的示例

这篇文章中的代码片段都是从 [示例](https://github.com/Scalingo/go-graceful-restart-example) 中提取的

## 结论

socket 传递配合 `ForkExec` 使用确实是一种无干扰更新进程的有效方式，在最大时间上，新的连接会等待几毫秒——用于服务的启动和恢复 `socket`，但这个时间很短。

这篇文章是我 \#周五技术系列 的一部分，下这个周不会有新的更新了，大家圣诞节快乐。

## References

* 示例: [https://github.com/Scalingo/go-graceful-restart-example](https://github.com/Scalingo/go-graceful-restart-example)
* Go变量的隐形声明: [http://www.qureet.com/blog/golang-beartrap/](http://www.qureet.com/blog/golang-beartrap/)
* Bonus的问题:fork\(\)Golang进程安全吗？ : [https://groups.google.com/forum/\#!msg/golang-nuts/beKfn7rujNg/zwY6zwl7QtQJ](https://groups.google.com/forum/#!msg/golang-nuts/beKfn7rujNg/zwY6zwl7QtQJ)
* 我们的网站: [https://appsdeck.eu](https://appsdeck.eu)

     — Léo Unbekandt CTO @ Appsdeck

* 译文地址：[https://www.oschina.net/translate/graceful-server-restart-with-go](https://www.oschina.net/translate/graceful-server-restart-with-go)
* 原文地址：[http://blog.appsdeck.eu/post/105609534953/graceful-server-restart-with-go](http://blog.appsdeck.eu/post/105609534953/graceful-server-restart-with-go)

