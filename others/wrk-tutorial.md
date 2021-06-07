---
description: wrk tutorial
---

# 性能测试工具wrk使用教程

被面试官经常问到之前开发的系统接口 QPS 能达到多少，经常给不出一个数值，支支吾吾，导致整体面试效果降低？

原因基本是一些公司中，做完功能测试就完了，压根不会有性能测试这一步，或者说并发量较少，没有必要进行性能测试，亦或者，交给测试人员后，只要整体问题不大，测试报告一般也是不会再给后端人员看的，这就导致我们在面试的时候，场面一度尴尬 ！！！

其实，不单单是针对面试，作为一名后端开发者，我们在完成一个接口开发后，在交给测试工程师之前，经常也会想知道，自己写的这个接口的性能如何呢？吞吐量能达到多少？QPS（Query per second 每秒处理完的请求数） 能达到多少呢？

这个时候，我们就需要借助一些常用的性能测试工具，如 Apache ab, Apache JMeter \(互联网公司用的较多\)，LoadRunner 等。

我们今天主要说一说轻量级性能测试工具 **wrk**。

## 一、什么是 wrk

摘自官方 GitHub 上的英文介绍：

> **wrk - a HTTP benchmarking tool**

> wrk is a modern HTTP benchmarking tool capable of generating significant load when run on a single multi-core CPU. It combines a multithreaded design with scalable event notification systems such as epoll and kqueue.

> An optional LuaJIT script can perform HTTP request generation, response processing, and custom reporting. Details are available in SCRIPTING and several examples are located in [scripts/](https://github.com/wg/wrk/blob/master/scripts).

**翻译一下：**

wrk 是一款针对 Http 协议的基准测试工具，它能够在单机多核 CPU 的条件下，使用系统自带的高性能 I/O 机制，如 epoll，kqueue 等，通过多线程和事件模式，对目标机器产生大量的负载。

> PS: 其实，wrk 是复用了 redis 的 ae 异步事件驱动框架，准确来说 ae 事件驱动框架并不是 redis 发明的, 它来至于 Tcl 的解释器 jim, 这个小巧高效的框架, 因为被 redis 采用而被大家所熟知。

## 二、 wrk 的优势&劣势

### 2.1 优势

* 轻量级性能测试工具;
* 安装简单（相对 Apache ab 来说）;
* 学习曲线基本为零，几分钟就能学会咋用了；
* 基于系统自带的高性能 I/O 机制，如 epoll, kqueue, 利用异步的事件驱动框架，通过很少的线程就可以压出很大的并发量；

### 2.2 劣势

wrk 目前仅支持单机压测，后续也不太可能支持多机器对目标机压测，因为它本身的定位，并不是用来取代 JMeter, LoadRunner 等专业的测试工具，wrk 提供的功能，对我们后端开发人员来说，应付日常接口性能验证还是比较友好的。

## 三、wrk 安装

wrk 只能被安装在类 Unix 系统上，所以我们需要一个 Linux 或者 MacOS 环境。Windows 10 安装需要开启自带的 Ubuntu 子系统。

### 3.1 Linux 安装

#### **3.1.1 Ubuntu/Debian**

依次执行如下命令：

```bash
sudo apt-get install build-essential libssl-dev git -y
git clone https://github.com/wg/wrk.git wrk
cd wrk
make
# 将可执行文件移动到 /usr/local/bin 位置
sudo cp wrk /usr/local/bin
```

#### **3.2.2 CentOS / RedHat / Fedora**

依次执行如下命令：

```bash
sudo yum groupinstall 'Development Tools'
sudo yum install -y openssl-devel git 
git clone https://github.com/wg/wrk.git wrk
cd wrk
make
# 将可执行文件移动到 /usr/local/bin 位置
sudo cp wrk /usr/local/bin
```

### 3.2 MacOS 安装

Mac 系统也可以通过先编译的方式来安装，但是更推荐使用 `brew` 的方式来安装, 步骤如下：

1. 安装 Homebrew，安装方式参考官网 [https://brew.sh](https://brew.sh/) （也就一行命令的事）;
2. 安装 wrk: `brew install wrk`;

### 3.3 Window 10 安装

Windown 10 需要在 `Windows 功能` 里勾选 `适用于 Linux 的 Windows 子系统`, 然后通过 `bash` 命令切换到 Ubuntu 子系统。接下来，参考 **3.1.1** Ubuntu 的操作系通中，安装 wrk 的步骤。

> 由于笔者使用的是 MacOS, Windows 上的安装步骤，并没有实际操作过，具体安装步骤，您可以参考官方 Windows 10 的安装教程：[https://github.com/wg/wrk/wiki/Installing-wrk-on-Windows-10](https://github.com/wg/wrk/wiki/Installing-wrk-on-Windows-10) ，或者用您喜欢的搜索引擎来搜索 Windows 10 如何启用 Ubuntu 子系统后，再安装 wrk，亦或者通过安装 Linux 虚拟机的方式来使用 wrk。

#### 3.4 验证一下，是否安装成功 <a id="34-&#x9A8C;&#x8BC1;&#x4E00;&#x4E0B;&#xFF0C;&#x662F;&#x5426;&#x5B89;&#x88C5;&#x6210;&#x529F;"></a>

命令行中输入命令：

```text
wrk -v
```

输出如上信息，说明安装成功了！

## 四、如何使用

安装成功了，要如何使用呢？

### 4.1 简单使用

```text
wrk -t12 -c400 -d30s http://www.baidu.com
```

这条命令表示，利用 wrk 对 `www.baidu.com` 发起压力测试，线程数为 12，模拟 400 个并发请求，持续 30 秒。

### 4.2 wrk 子命令参数说明

除了上面简单示例中使用到的子命令参数，`wrk` 还有其他更丰富的功能，命令行中输入 `wrk --help`, 可以看到支持以下子命令：

```bash
[root@VM_0_5_centos ~]# wrk --help
Usage: wrk <options> <url>                            
  Options:                                            
    -c, --connections <N>  Connections to keep open   
    -d, --duration    <T>  Duration of test           
    -t, --threads     <N>  Number of threads to use   
                                                      
    -s, --script      <S>  Load Lua script file       
    -H, --header      <H>  Add header to request      
        --latency          Print latency statistics   
        --timeout     <T>  Socket/request timeout     
    -v, --version          Print version details      
                                                      
  Numeric arguments may include a SI unit (1k, 1M, 1G)
  Time arguments may include a time unit (2s, 2m, 2h)
```

翻译一下：

```bash
使用方法: wrk <选项> <被测HTTP服务的URL>                            
  Options:                                            
    -c, --connections <N>  跟服务器建立并保持的TCP连接数量  
    -d, --duration    <T>  压测时间           
    -t, --threads     <N>  使用多少个线程进行压测   
                                                      
    -s, --script      <S>  指定Lua脚本路径       
    -H, --header      <H>  为每一个HTTP请求添加HTTP头      
        --latency          在压测结束后，打印延迟统计信息   
        --timeout     <T>  超时时间     
    -v, --version          打印正在使用的wrk的详细版本信息
                                                      
  <N>代表数字参数，支持国际单位 (1k, 1M, 1G)
  <T>代表时间参数，支持时间单位 (2s, 2m, 2h)
```

> PS: 关于线程数，并不是设置的越大，压测效果越好，线程设置过大，反而会导致线程切换过于频繁，效果降低，一般来说，推荐设置成压测机器 CPU 核心数的 2 倍到 4 倍就行了。

### 4.3 测试报告

执行压测命令:

```bash
wrk -t12 -c400 -d30s --latency http://www.baidu.com
```

执行上面的压测命令，30 秒压测过后，生成如下压测报告：

```bash
Running 30s test @ http://www.baidu.com 
  12 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   386.32ms  380.75ms   2.00s    86.66%
    Req/Sec    17.06     13.91   252.00     87.89%
  Latency Distribution
     50%  218.31ms
     75%  520.60ms
     90%  955.08ms
     99%    1.93s 
  4922 requests in 30.06s, 73.86MB read
  Socket errors: connect 0, read 0, write 0, timeout 311
Requests/sec:    163.76
Transfer/sec:      2.46MB
```

我们来具体说一说，报告中各项指标都代表什么意思：

```bash
Running 30s test @ http://www.baidu.com （压测时间30s）
  12 threads and 400 connections （共12个测试线程，400个连接）
			  （平均值） （标准差）  （最大值）（正负一个标准差所占比例）
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    （延迟）
    Latency   386.32ms  380.75ms   2.00s    86.66%
    (每秒请求数)
    Req/Sec    17.06     13.91   252.00     87.89%
  Latency Distribution （延迟分布）
     50%  218.31ms
     75%  520.60ms
     90%  955.08ms
     99%    1.93s 
  4922 requests in 30.06s, 73.86MB read (30.06s内处理了4922个请求，耗费流量73.86MB)
  Socket errors: connect 0, read 0, write 0, timeout 311 (发生错误数)
Requests/sec:    163.76 (QPS 163.76,即平均每秒处理请求数为163.76)
Transfer/sec:      2.46MB (平均每秒流量2.46MB)
```

可以看到，压测报告还是非常直观的！

> **标准差**啥意思？标准差如果太大说明样本本身离散程度比较高，有可能系统性能波动较大。

## 五 使用 Lua 脚本进行复杂测试

您可能有疑问了，你这种进行 GET 请求还凑合，我想进行 POST 请求咋办？而且我想每次的请求参数都不一样，用来模拟用户使用的实际场景，又要怎么弄呢？

对于这种需求，我们可以通过编写 Lua 脚本的方式，在运行压测命令时，通过参数 `--script` 来指定 Lua 脚本，来满足个性化需求。

```text
/wrk -t1 -c10 -d20s -s post.lua --latency http://www.douban.com
```

### **5.1 wrk 对 Lua 脚本的支持**

wrk 支持在三个阶段对压测进行个性化，分别是启动阶段、运行阶段和结束阶段。每个测试线程，都拥有独立的Lua 运行环境。

#### **启动阶段:**

```lua
function setup(thread)
```

在脚本文件中实现 setup 方法，wrk 就会在测试线程已经初始化，但还没有启动的时候调用该方法。wrk会为每一个测试线程调用一次 setup 方法，并传入代表测试线程的对象 thread 作为参数。setup 方法中可操作该 thread 对象，获取信息、存储信息、甚至关闭该线程。

```lua
thread.addr             - get or set the thread's server address
thread:get(name)        - get the value of a global in the thread's env
thread:set(name, value) - set the value of a global in the thread's env
thread:stop()           - stop the thread
```

#### **运行阶段：**

```lua
function init(args)
function delay()
function request()
function response(status, headers, body)
```

* `init(args)`: 由测试线程调用，只会在进入运行阶段时，调用一次。支持从启动 wrk 的命令中，获取命令行参数；
* `delay()`： 在每次发送请求之前调用，如果需要定制延迟时间，可以在这个方法中设置；
* `request()`: 用来生成请求, 每一次请求都会调用该方法，所以注意不要在该方法中做耗时的操作；
* `response(status, headers, body)`: 在每次收到一个响应时被调用，为提升性能，如果没有定义该方法，那么wrk不会解析 `headers` 和 `body`；

#### **结束阶段：**

```text
function done(summary, latency, requests)
```

done\(\) 方法在整个测试过程中只会被调用一次，我们可以从给定的参数中，获取压测结果，生成定制化的测试报告。

**自定义 Lua 脚本中可访问的变量以及方法：**

变量：wrk

```text
wrk = {
    scheme  = "http",
    host    = "localhost",
    port    = 8080,
    method  = "GET",
    path    = "/",
    headers = {},
    body    = nil,
    thread  = <userdata>,
  }
```

以上定义了一个 `table` 类型的全局变量，修改该 `wrk` 变量，会影响所有请求。

方法：

1. `wrk.fomat`
2. `wrk.lookup`
3. `wrk.connect`

上面三个方法解释如下：

```text
function wrk.format(method, path, headers, body)

    wrk.format returns a HTTP request string containing the passed parameters
    merged with values from the wrk table.
    # 根据参数和全局变量 wrk，生成一个 HTTP rquest 字符串。

function wrk.lookup(host, service)

    wrk.lookup returns a table containing all known addresses for the host
    and service pair. This corresponds to the POSIX getaddrinfo() function.
    # 给定 host 和 service（port/well known service name），返回所有可用的服务器地址信息。

function wrk.connect(addr)

    wrk.connect returns true if the address can be connected to, otherwise
    it returns false. The address must be one returned from wrk.lookup().
    # 测试给定的服务器地址信息是否可以成功创建连接
```

### **5.2 调用 POST 接口**

{% code title="post.lua" %}
```lua
wrk.method = "POST"
wrk.body   = "foo=bar&baz=quux"
wrk.headers["Content-Type"] = "application/x-www-form-urlencoded"
```
{% endcode %}

注意: wrk 是个全局变量，这里对其做了修改，使得所有请求都使用 POST 的方式，并指定了 body 和 Content-Type头。

调用

```bash
./wrk -t4 -c100 -d30s -T30s --script=post.lua --latency http://www.douban.com
```

### **5.3 自定义每次请求的参数**

```lua
request = function()
   uid = math.random(1, 10000000)
   path = "/test?uid=" .. uid
   return wrk.format(nil, path)
end
```

在 request 方法中，随机生成 1~10000000 之间的 uid，并动态生成请求 URL.

### **5.4 每次请求前，延迟 10ms**

```lua
function delay()
   return 10
end
```

### **5.5 认证**

**请求的接口需要先进行认证，获取 token 后，才能发起请求，咋办？**

```lua
token = nil
path  = "/auth"

request = function()
   return wrk.format("GET", path)
end

response = function(status, headers, body)
   if not token and status == 200 then
      token = headers["X-Token"]
      path  = "/test"
      wrk.headers["X-Token"] = token
   end
end
```

上面的脚本表示，在 token 为空的情况下，先请求 `/auth` 接口来认证，获取 token, 拿到 token 以后，将 token 放置到请求头中，再请求真正需要压测的 `/test` 接口。

### **5.6 压测支持 HTTP pipeline 的服务**

```lua
init = function(args)
   local r = {}
   r[1] = wrk.format(nil, "/?foo")
   r[2] = wrk.format(nil, "/?bar")
   r[3] = wrk.format(nil, "/?baz")

   req = table.concat(r)
end

request = function()
   return req
end
```

通过在 init 方法中将三个 HTTP请求拼接在一起，实现每次发送三个请求，以使用 HTTP pipeline。

### 5.7 使用lua实现文件上传

```lua
# 实现上传文件测试与json类似
# 同样是设置wrk.body和wrk.headers的值, 只是body较麻烦一些
wrk.method = "POST"
wrk.headers["Content-Type"] = "multipart/form-data;boundary=------WebKitFormBoundaryX3bY6PBMcxB1vCan"

file = io.open("path/to/fake.jpg", "rb")

-- 拼装form-data
form = "------WebKitFormBoundaryX3bY6PBMcxB1vCan\r\n"
form = form .. "Content-Disposition: form-data; name="file"; filename="fake.jpg"\r\n"
form = form .. "Content-Type: image/jpeg\r\n\r\n"
form = form .. file:read("*a")
form = form .. "\r\n------WebKitFormBoundaryX3bY6PBMcxB1vCan--"

wrk.body  = form
```

## References

* 原文 [性能测试工具 wrk 使用教程](https://www.cnblogs.com/quanxiaoha/p/10661650.html)
* [小巧而强大的wrk压测工具](https://www.escapelife.site/posts/4b014d0b.html)
* [https://github.com/wg/wrk](https://github.com/wg/wrk)
* [https://zjumty.iteye.com/blog/2221040](https://zjumty.iteye.com/blog/2221040)
* [http://www.cnblogs.com/xinzhao/p/6233009.html](http://www.cnblogs.com/xinzhao/p/6233009.html)

