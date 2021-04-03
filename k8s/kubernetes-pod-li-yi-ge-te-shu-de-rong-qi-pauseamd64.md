# Kubernetes pod里一个特殊的容器：pause-amd64

大家在使用Docker容器或者Kubernetes时,遇到过这个容器么?`gcr.io/google_containers/pause-amd64`

`docker ps`的命令返回的结果：

```bash
[root@k8s-minion1 kubernetes]# docker ps |grep pause
c3026adee957        gcr.io/google_containers/pause-amd64:3.0           "/pause"                 22 minutes ago      Up 22 minutes                           k8s_POD.d8dbe16c_redis-master-343230949-04glm_default_ce3f60a9-095d-11e7-914b-0a77ecd65f3e_66c108d5
202df18d636e        gcr.io/google_containers/pause-amd64:3.0           "/pause"                 24 hours ago        Up 24 hours                             k8s_POD.d8dbe16c_kube-proxy-js0z0_kube-system_2866cfc2-0891-11e7-914b-0a77ecd65f3e_c8e1a667
072d3414d33a        gcr.io/google_containers/pause-amd64:3.0           "/pause"                 24 hours ago        Up 24 hours                             k8s_POD.d8dbe16c_kube-flannel-ds-tsps5_default_2866e3fb-0891-11e7-914b-0a77ecd65f3e_be4b719e
[root@k8s-minion1 kubernetes]#
```

## 用途

Kubernetes的官网解释：

{% hint style="info" %}
it’s part of the infrastructure. This container is started first in all Pods to setup the network for the Pod.
{% endhint %}

意思是：`pause-amd64`是Kubernetes基础设施的一部分，Kubernetes管理的所有pod里，`pause-amd64`容器是第一个启动的，用于实现Kubernetes集群里pod之间的网络通讯。

对这个特殊容器感兴趣的朋友，可以阅读其源代码： [https://github.com/kubernetes/kubernetes/tree/master/build/pause](https://github.com/kubernetes/kubernetes/tree/master/build/pause)

## Dockerfile

我们查看这个pause-amd64镜像的dockerfile，发现实现很简单，基于一个空白镜像开始：

```bash
FROM scratch
ARG ARCH
ADD bin/pause-${ARCH} /pause
ENTRYPOINT ["/pause"]
```

`ARG`指令用于指定在执行`docker build`命令时传递进去的参数。

这个pause container是用C语言写的：[https://www.ianlewis.org/en/almighty-pause-container](https://www.ianlewis.org/en/almighty-pause-container)

在运行的Kubernetes node上运行`docker ps`，能发现这些`pause container`：

![](../.gitbook/assets/image%20%2821%29.png)

## 功能

`pause container`作为`pod`里其他所有`container`的`parent container`，主要有两个职责：

* 是`pod`里其他容器共享`Linux namespace`的基础 
* 扮演`PID 1`的角色，负责处理僵尸进程

这两点我会逐一细说。

### 共享namespace

在Linux里，当父进程fork一个新进程时，子进程会从父进程继承`namespace`。目前Linux实现了六种类型的`namespace`，每一个`namespace`是包装了一些全局系统资源的抽象集合，这一抽象集合使得在进程的命名空间中可以看到全局系统资源。命名空间的一个总体目标是支持轻量级虚拟化工具`container`的实现，`container`机制本身对外提供一组进程，这组进程自己会认为它们就是系统唯一存在的进程。

在Linux里，父进程`fork`的子进程会继承父进程的命名空间。与这种行为相反的一个系统命令就是`unshare`：

![](../.gitbook/assets/image%20%2826%29.png)

### 处理僵尸进程

再来聊聊`pause`容器如何处理僵尸进程的。

`Pause`容器内其实就运行了一个非常简单的进程，其逻辑可以从前面提到的Pause [github仓库](https://github.com/kubernetes/kubernetes/tree/master/build/pause)上找到：

```java
static void sigdown(int signo) {
  psignal(signo, "Shutting down, got signal");
  exit(0);
}

static void sigreap(int signo) {
  while (waitpid(-1, NULL, WNOHANG) > 0);
}

int main() {
  if (getpid() != 1)
    /* Not an error because pause sees use outside of infra containers. */
    fprintf(stderr, "Warning: pause should be the first process\n");

  if (sigaction(SIGINT, &(struct sigaction){.sa_handler = sigdown}, NULL) < 0)
    return 1;
  if (sigaction(SIGTERM, &(struct sigaction){.sa_handler = sigdown}, NULL) < 0)
    return 2;
  if (sigaction(SIGCHLD, &(struct sigaction){.sa_handler = sigreap,
                                             .sa_flags = SA_NOCLDSTOP},
                NULL) < 0)
    return 3;

  for (;;)
    pause();
  fprintf(stderr, "Error: infinite loop terminated\n");
  return 42;
}
```

这个c语言实现的进程，核心代码就28行， 其中第24行里一个无限循环`for(;;)`, 至此大家能看出来pause容器名称的由来了吧？这个无限循环里执行的是一个系统调用`pause`，

```java
PAUSE(2)                Linux Programmer's Manual               PAUSE(2)
NAME         top
       pause - wait for signal
SYNOPSIS         top
       #include <unistd.h>

       int pause(void);
DESCRIPTION         top
       pause() causes the calling process (or thread) to sleep until a
       signal is delivered that either terminates the process or causes
       the invocation of a signal-catching function.
RETURN VALUE         top
       pause() returns only when a signal was caught and the signal-
       catching function returned.  In this case, pause() returns -1,
       and errno is set to EINTR.
ERRORS         top
       EINTR  a signal was caught and the signal-catching function
              returned.
CONFORMING TO         top
       POSIX.1-2001, POSIX.1-2008, SVr4, 4.3BSD.
SEE ALSO         top
       kill(2), select(2), signal(2), sigsuspend(2)
COLOPHON         top
       This page is part of release 5.11 of the Linux man-pages project.
       A description of the project, information about reporting bugs,
       and the latest version of this page, can be found at
       https://www.kernel.org/doc/man-pages/.
```

因此`pause`容器大部分时间都在沉睡，等待有信号将其唤醒。接收什么信号呢？

一旦收到`SIGCHLD`信号，`pause`进程就执行注册的`sigreap`函数。

![](../.gitbook/assets/image%20%2825%29.png)

看下`SIGCHLD`信号的帮助：

![](../.gitbook/assets/image%20%2824%29.png)

`SIGCHLD`，在一个进程正常终止或者停止时，将`SIGCHLD`信号发送给其父进程，按系统默认将忽略此信号，如果父进程希望被告知其子系统的这种状态，则应捕捉此信号。

pause进程注册的信号处理函数`sigreap`里，调用另一个系统调用`waitpid`来获得子进程终止的原因。

![](../.gitbook/assets/image%20%2822%29.png)

希望这篇文章对大家理解Kubernetes里的pause容器有所帮助。感谢阅读。

## References

* [原文 Kubernetes pod里一个特殊的容器：pause-amd64](https://blog.csdn.net/i042416/article/details/85160895) ,  [汪子熙](https://jerry.blog.csdn.net/)

