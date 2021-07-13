# 【k8s】leaderelection选举策略实现组件高可用

##  一、前言

分布式服务绕不开一个词：选主。今天主要来聊一下k8s中是如何通过leaderelection来实现组件的高可用的。在k8s本身的组件中，kube-scheduler和kube-manager-controller两个组件是有leader选举的，这个选举机制是k8s对于这两个组件的高可用保障。即正常情况下kube-scheduler或kube-manager-controller组件的多个副本只有一个是处于业务逻辑运行状态，其它副本则不断的尝试去获取锁，去竞争leader，直到自己成为leader。如果正在运行的leader因某种原因导致当前进程退出，或者锁丢失，则由其它副本去竞争新的leader，获取leader继而执行业务逻辑。

不光是k8s本身组件用到了这个选举策略，我们自己定义的服务同样可以用这个算法去实现选主。在k8s client包中就提供了接口供用户使用。代码路径在client-go/tools/leaderelection下。今天先说怎么用，再问为什么？

## 二、leaderelection使用demo

###  1. demo代码

```go

/*
例子来源于client-go中的样例
*/
package main
import (
	"context"
	"flag"
	"os"
	"os/signal"
	"syscall"
	"time"
 
	"github.com/google/uuid"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	clientset "k8s.io/client-go/kubernetes"
	"k8s.io/client-go/rest"
	"k8s.io/client-go/tools/clientcmd"
	"k8s.io/client-go/tools/leaderelection"
	"k8s.io/client-go/tools/leaderelection/resourcelock"
	"k8s.io/klog"
)
 
func buildConfig(kubeconfig string) (*rest.Config, error) {
	if kubeconfig != "" {
		cfg, err := clientcmd.BuildConfigFromFlags("", kubeconfig)
		if err != nil {
			return nil, err
		}
		return cfg, nil
	}
 
	cfg, err := rest.InClusterConfig()
	if err != nil {
		return nil, err
	}
	return cfg, nil
}
 
func main() {
	klog.InitFlags(nil)
 
	var kubeconfig string
	var leaseLockName string
	var leaseLockNamespace string
	var id string
 
	flag.StringVar(&kubeconfig, "kubeconfig", "", "absolute path to the kubeconfig file")
	flag.StringVar(&id, "id", uuid.New().String(), "the holder identity name")
	flag.StringVar(&leaseLockName, "lease-lock-name", "", "the lease lock resource name")
	flag.StringVar(&leaseLockNamespace, "lease-lock-namespace", "", "the lease lock resource namespace")
	flag.Parse()
 
	if leaseLockName == "" {
		klog.Fatal("unable to get lease lock resource name (missing lease-lock-name flag).")
	}
	if leaseLockNamespace == "" {
		klog.Fatal("unable to get lease lock resource namespace (missing lease-lock-namespace flag).")
	}
 
	// leader election uses the Kubernetes API by writing to a
	// lock object, which can be a LeaseLock object (preferred),
	// a ConfigMap, or an Endpoints (deprecated) object.
	// Conflicting writes are detected and each client handles those actions
	// independently.
	config, err := buildConfig(kubeconfig)
	if err != nil {
		klog.Fatal(err)
	}
	client := clientset.NewForConfigOrDie(config)
 
	run := func(ctx context.Context) {
		// complete your controller loop here
		klog.Info("Controller loop...")
 
		select {}
	}
 
	// use a Go context so we can tell the leaderelection code when we
	// want to step down
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
 
	// listen for interrupts or the Linux SIGTERM signal and cancel
	// our context, which the leader election code will observe and
	// step down
	ch := make(chan os.Signal, 1)
	signal.Notify(ch, os.Interrupt, syscall.SIGTERM)
	go func() {
		<-ch
		klog.Info("Received termination, signaling shutdown")
		cancel()
	}()
 
	// we use the Lease lock type since edits to Leases are less common
	// and fewer objects in the cluster watch "all Leases".
    // 指定锁的资源对象，这里使用了Lease资源，还支持configmap，endpoint，或者multilock(即多种配合使用)
	lock := &resourcelock.LeaseLock{
		LeaseMeta: metav1.ObjectMeta{
			Name:      leaseLockName,
			Namespace: leaseLockNamespace,
		},
		Client: client.CoordinationV1(),
		LockConfig: resourcelock.ResourceLockConfig{
			Identity: id,
		},
	}
 
	// start the leader election code loop
	leaderelection.RunOrDie(ctx, leaderelection.LeaderElectionConfig{
		Lock: lock,
		// IMPORTANT: you MUST ensure that any code you have that
		// is protected by the lease must terminate **before**
		// you call cancel. Otherwise, you could have a background
		// loop still running and another process could
		// get elected before your background loop finished, violating
		// the stated goal of the lease.
		ReleaseOnCancel: true,
		LeaseDuration:   60 * time.Second,//租约时间
		RenewDeadline:   15 * time.Second,//更新租约的
		RetryPeriod:     5 * time.Second,//非leader节点重试时间
		Callbacks: leaderelection.LeaderCallbacks{
			OnStartedLeading: func(ctx context.Context) {
                //变为leader执行的业务代码
				// we're notified when we start - this is where you would
				// usually put your code
				run(ctx)
			},
			OnStoppedLeading: func() {
                 // 进程退出
				// we can do cleanup here
				klog.Infof("leader lost: %s", id)
				os.Exit(0)
			},
			OnNewLeader: func(identity string) {
                //当产生新的leader后执行的方法
				// we're notified when new leader elected
				if identity == id {
					// I just got the lock
					return
				}
				klog.Infof("new leader elected: %s", identity)
			},
		},
	})
}
```

### 2. 启动进程1

```text
go run main.go -kubeconfig=/tangqing2/.kube/config -logtostderr=true -lease-lock-name=example -lease-lock-namespace=default -id=1 -v=4
I0215 14:56:37.049658   48045 leaderelection.go:242] attempting to acquire leader lease  default/example...
I0215 14:56:37.080368   48045 leaderelection.go:252] successfully acquired lease default/example
I0215 14:56:37.080437   48045 main.go:87] Controller loop...
```

### 3. 启动进程2

```bash
➜  leaderelection git:(master) ✗ go run main.go -kubeconfig=/tangqing2/.kube/config -logtostderr=true -lease-lock-name=example -lease-lock-namespace=default -id=2 -v=4
I0215 15:57:35.870051   48791 leaderelection.go:242] attempting to acquire leader lease  default/example...
I0215 15:57:35.894735   48791 leaderelection.go:352] lock is held by 1 and has not yet expired
I0215 15:57:35.894769   48791 leaderelection.go:247] failed to acquire lease default/example
I0215 15:57:35.894790   48791 main.go:151] new leader elected: 1
I0215 15:57:44.532991   48791 leaderelection.go:352] lock is held by 1 and has not yet expired
I0215 15:57:44.533028   48791 leaderelection.go:247] failed to acquire lease default/example
```

这里可以看出来id=1的进程持有锁，并且运行的程序，而id=2的进程表示无法获取到锁，在不断的进程尝试。

现在kill掉id=1进程，在等待lock释放之后（有个LeaseDuration时间），leader变为id=2的进程执行工作

```bash
I0215 20:01:41.489300   48791 leaderelection.go:252] successfully acquired lease default/example
I0215 20:01:41.489577   48791 main.go:87] Controller loop...
```

## 三、深入理解\(源码分析\)

 第二章的demo基本原理其实就是利用通过Kubernetes中 configmap ， endpoints 或者 lease 资源实现一个分布式锁，抢\(acqure\)到锁的节点成为leader，并且定期更新（renew）。其他进程也在不断的尝试进行抢占，抢占不到则继续等待下次循环。当leader节点挂掉之后，租约到期，其他节点就成为新的leader。

代码路径在client-go/tools/leaderelection下.逻辑结构如下图：

![](../.gitbook/assets/image%20%2851%29.png)

### Interface接口

**Interface**: 中定义了一系列方法, 包括增加修改获取一个LeaderElectionRecord, 说白了就是一个客户端, 而且每个客户端实例都要有自己分布式唯一的id.

```go
// tools/leaderelection/resourcelock/interface.go
 
type LeaderElectionRecord struct {
    // 持有者的id 也就是leader的id
    HolderIdentity       string      `json:"holderIdentity"`
    // 一个租约多长时间
    LeaseDurationSeconds int         `json:"leaseDurationSeconds"`
    // 获得leader的时间
    AcquireTime          metav1.Time `json:"acquireTime"`
    // 续约的时间
    RenewTime            metav1.Time `json:"renewTime"`
    // leader变更的次数
    LeaderTransitions    int         `json:"leaderTransitions"`
}
 
type Interface interface {
    // 返回当前资源LeaderElectionRecord 
    Get() (*LeaderElectionRecord, error)
    // 创建一个资源LeaderElectionRecord
    Create(ler LeaderElectionRecord) error
    // 更新资源
    Update(ler LeaderElectionRecord) error
    // 记录事件
    RecordEvent(string)
    // 返回当前该应用的id
    Identity() string
    // 描述信息
    Describe() string
}
```

它有三个实现类, 分别为EndpointLock, ConfigMapLock和LeaseLock分别可以操作k8s中的endpoint, configmap和lease. 也就是提供了这三种资源类型. 这里以EndpointLock为例子说明.

```go
// tools/leaderelection/resourcelock/endpointslock.go
type EndpointsLock struct {
    // 必须包括namespace和name
    EndpointsMeta metav1.ObjectMeta
    // 访问api-server的客户端
    Client        corev1client.EndpointsGetter
    // 该EndpointsLock的分布式唯一身份id
    LockConfig    ResourceLockConfig
    // 当前操作的endpoint
    e             *v1.Endpoints
}
// tools/leaderelection/resourcelock/interface.go
type ResourceLockConfig struct {
// 分布式唯一id
    Identity string
    EventRecorder EventRecorder
}
```

Create, Update, Get方法都是利用client去访问k8s的api-server

```go
// tools/leaderelection/resourcelock/endpointslock.go
 
func (el *EndpointsLock) Get() (*LeaderElectionRecord, error) {
    var record LeaderElectionRecord
    var err error
    el.e, err = el.Client.Endpoints(el.EndpointsMeta.Namespace).Get(el.EndpointsMeta.Name, metav1.GetOptions{})
    if err != nil {
        return nil, err
    }
    if el.e.Annotations == nil {
        el.e.Annotations = make(map[string]string)
    }
    if recordBytes, found := el.e.Annotations[LeaderElectionRecordAnnotationKey]; found {
        if err := json.Unmarshal([]byte(recordBytes), &record); err != nil {
            return nil, err
        }
    }
    return &record, nil
}
 
// Create attempts to create a LeaderElectionRecord annotation
func (el *EndpointsLock) Create(ler LeaderElectionRecord) error {
    recordBytes, err := json.Marshal(ler)
    if err != nil {
        return err
    }
    el.e, err = el.Client.Endpoints(el.EndpointsMeta.Namespace).Create(&v1.Endpoints{
        ObjectMeta: metav1.ObjectMeta{
            Name:      el.EndpointsMeta.Name,
            Namespace: el.EndpointsMeta.Namespace,
            Annotations: map[string]string{
                LeaderElectionRecordAnnotationKey: string(recordBytes),
            },
        },
    })
    return err
}
```

另外interface还提供了生成各个子类的方法

```go
func New(lockType string, ns string, name string, coreClient corev1.CoreV1Interface, coordinationClient coordinationv1.CoordinationV1Interface, rlc ResourceLockConfig) (Interface, error) {
    switch lockType {
    case EndpointsResourceLock:
        return &EndpointsLock{
            EndpointsMeta: metav1.ObjectMeta{
                Namespace: ns,
                Name:      name,
            },
            Client:     coreClient,
            LockConfig: rlc,
        }, nil
    ...
    default:
        return nil, fmt.Errorf("Invalid lock-type %s", lockType)
    }
}
```

### 2. LeaderElector

LeaderElector是一个竞争资源的实体.

```go
type LeaderElectionConfig struct {
    // 客户端
    Lock rl.Interface
    LeaseDuration time.Duration
    RenewDeadline time.Duration
    RetryPeriod time.Duration
    // 需要用户配置的回调函数
    Callbacks LeaderCallbacks
    WatchDog *HealthzAdaptor
    // 判断在cancel的时候如果当前是leader是否需要释放
    ReleaseOnCancel bool
    Name string
}
```

LeaderElectionConfig拥有一个Interface对象, 以及用户需要配置的回调函数LeaderCallbacks对象. 关于LeaseDuration, RenewDeadline和RetryPeriod会在方法中介绍.

```go

type LeaderElector struct {
    // 用于保存当前应用的一些配置 包括该应用的id等等
    config LeaderElectionConfig
    // 远程获取的资源 (不一定自己是leader) 所有想竞争此资源的应用获取的是同一份
    observedRecord rl.LeaderElectionRecord
    // 获取的时间
    observedTime   time.Time
    reportedLeader string
    clock clock.Clock
    metrics leaderMetricsAdapter
    name string
}
```

这里着重要关注以下几个属性: 

* config: 该LeaderElectionConfig对象配置了当前应用的客户端, 以及此客户端的唯一id等等.
* observedRecord: 该LeaderElectionRecord就是保存着从api-server中获得的leader的信息
* observedTime: 获得的时间.

很明显判断当前进程是不是leader只需要判断config中的id和observedRecord中的id是不是一致即可.

```go
func (le *LeaderElector) GetLeader() string {
    return le.observedRecord.HolderIdentity
}
 
// IsLeader returns true if the last observed leader was this client else returns false.
func (le *LeaderElector) IsLeader() bool {
    return le.observedRecord.HolderIdentity == le.config.Lock.Identity()
}
```

### 3. RUN

```go
func (le *LeaderElector) Run(ctx context.Context) {
    defer func() {
        runtime.HandleCrash()
        le.config.Callbacks.OnStoppedLeading()
    }()
    // 如果获取失败 那就是ctx signalled done
    // 不然即使失败, 该client也会一直去尝试获得leader位置
    if !le.acquire(ctx) {
        return // ctx signalled done
    }
    // 如果获得leadership 以goroutine和回调的形式启动用户自己的逻辑方法OnStartedLeading
    ctx, cancel := context.WithCancel(ctx)
    defer cancel()
    go le.config.Callbacks.OnStartedLeading(ctx)
    // 一直去续约 这里也是一个循环操作
    // 如果失去了leadership 该方法才会返回
    // 该方法返回 整个Run方法就返回了
    le.renew(ctx)
}

```

1. 该client\(也就是le这个实例\)首先会调用acquire方法一直尝试去竞争leadership. \(如果竞争失败, 继续竞争, 不会进入2。 竞争成功, 进入2；
2. 异步启动用户自己的逻辑程序\(OnStartedLeading\). 进入3
3. 通过调用renew方法续约自己的leadership. 续约成功, 继续续约. 续约失败, 整个Run就结束了.

### 4.acquire

```text
func (le *LeaderElector) maybeReportTransition() {
    // 如果没有变化 则不需要更新
    if le.observedRecord.HolderIdentity == le.reportedLeader {
        return
    }
    // 更新reportedLeader为最新的leader的id
    le.reportedLeader = le.observedRecord.HolderIdentity
    if le.config.Callbacks.OnNewLeader != nil {
        // 调用当前应用的回调函数OnNewLeader报告新的leader产生
        go le.config.Callbacks.OnNewLeader(le.reportedLeader)
    }
}
 
// 一旦获得leadership 立马返回true
// 返回false的唯一情况是ctx signals done
func (le *LeaderElector) acquire(ctx context.Context) bool {
    ctx, cancel := context.WithCancel(ctx)
    defer cancel()
    succeeded := false
    desc := le.config.Lock.Describe()
    klog.Infof("attempting to acquire leader lease  %v...", desc)
    wait.JitterUntil(func() {
        // 尝试获得或者更新资源
        succeeded = le.tryAcquireOrRenew()
        // 有可能会产生新的leader
        // 所以调用maybeReportTransition检查是否需要广播新产生的leader
        le.maybeReportTransition()
        if !succeeded {
            // 如果获得leadership失败 则返回后继续竞争
            klog.V(4).Infof("failed to acquire lease %v", desc)
            return
        }
        // 自己成为leader
        // 可以调用cancel方法退出JitterUntil进而从acquire中返回
        le.config.Lock.RecordEvent("became leader")
        le.metrics.leaderOn(le.config.Name)
        klog.Infof("successfully acquired lease %v", desc)
        cancel()
    }, le.config.RetryPeriod, JitterFactor, true, ctx.Done())
    return succeeded
}
```

 **acquire** 的作用如下: 

1. 一旦获得leadership, 立马返回true. 否则会隔RetryPeriod时间尝试一次
2. 一旦有ctx signals done, 会返回false

这里的逻辑比较简单, 主要的逻辑是在tryAcquireOrRenew方法中.

### 5. renew

```go
// RenewDeadline=10s RetryPeriod=2s
func (le *LeaderElector) renew(ctx context.Context) {
    ctx, cancel := context.WithCancel(ctx)
    defer cancel()
    // 每隔RetryPeriod会调用 除非cancel()方法被调用才会退出
    wait.Until(func() {
        timeoutCtx, timeoutCancel := context.WithTimeout(ctx, le.config.RenewDeadline)
        defer timeoutCancel()
        // 每隔2ms调用该方法直到该方法返回true为止
        // 如果超时了也会退出该方法 并且err中有错误信息
        err := wait.PollImmediateUntil(le.config.RetryPeriod, func() (bool, error) {
            done := make(chan bool, 1)
            go func() {
                defer close(done)
                done <- le.tryAcquireOrRenew()
            }()
 
            select {
            case <-timeoutCtx.Done():
                return false, fmt.Errorf("failed to tryAcquireOrRenew %s", timeoutCtx.Err())
            case result := <-done:
                return result, nil
            }
        }, timeoutCtx.Done())
 
        // 有可能会产生新的leader 如果有会广播新产生的leader
        le.maybeReportTransition()
        desc := le.config.Lock.Describe()
        if err == nil {
            // 如果err == nil, 表明上面PollImmediateUntil中返回true了 续约成功 依然处于leader位置
            // 返回后 继续运行wait.Until的逻辑
            klog.V(5).Infof("successfully renewed lease %v", desc)
            return
        }
        // err != nil 表明超时了 试的总时间超过了RenewDeadline 失去了leader位置 续约失败
        // 调用cancel方法退出wait.Until
        le.config.Lock.RecordEvent("stopped leading")
        le.metrics.leaderOff(le.config.Name)
        klog.Infof("failed to renew lease %v: %v", desc, err)
        cancel()
    }, le.config.RetryPeriod, ctx.Done())
 
    // if we hold the lease, give it up
    if le.config.ReleaseOnCancel {
        le.release()
    }
}
```

> 可以看到该client的base条件是它自己是当前的leader, 然后来续约操作.

> 这里来说一下RenewDeadline和RetryPeriod的作用.
>
>  **每隔RetryPeriod时间会通过tryAcquireOrRenew续约, 如果续约失败, 还会进行再次尝试. 一直到尝试的总时间超过RenewDeadline后该client就会失去leadership.**

### 6. tryAcquireOrRenew

```go
// 竞争或者更新leadership
// 成功返回true 失败返回false
func (le *LeaderElector) tryAcquireOrRenew() bool {
    now := metav1.Now()
    leaderElectionRecord := rl.LeaderElectionRecord{
        HolderIdentity:       le.config.Lock.Identity(),
        LeaseDurationSeconds: int(le.config.LeaseDuration / time.Second),
        RenewTime:            now,
        AcquireTime:          now,
    }
 
    // 1. obtain or create the ElectionRecord
    // 从client端中获得ElectionRecord
    oldLeaderElectionRecord, err := le.config.Lock.Get()
    if err != nil {
        if !errors.IsNotFound(err) {
            // 失败直接退出
            klog.Errorf("error retrieving resource lock %v: %v", le.config.Lock.Describe(), err)
            return false
        }
        // 因为没有获取到, 因此创建一个新的进去
        if err = le.config.Lock.Create(leaderElectionRecord); err != nil {
            klog.Errorf("error initially creating leader election record: %v", err)
            return false
        }
        // 然后设置observedRecord为刚刚加入进去的leaderElectionRecord
        le.observedRecord = leaderElectionRecord
        le.observedTime = le.clock.Now()
        return true
    }
 
    // 2. Record obtained, check the Identity & Time
    // 从远端获取到record(资源)成功存到oldLeaderElectionRecord
    // 如果oldLeaderElectionRecord与observedRecord不相同 更新observedRecord
    // 因为observedRecord代表是从远端存在Record
 
    // 需要注意的是每个client都在竞争leadership, 而leader一直在续约, leader会更新它的RenewTime字段
    // 所以一旦leader续约成功 每个non-leader候选者都需要更新其observedTime和observedRecord
    if !reflect.DeepEqual(le.observedRecord, *oldLeaderElectionRecord) {
        le.observedRecord = *oldLeaderElectionRecord
        le.observedTime = le.clock.Now()
    }
    // 如果leader已经被占有并且不是当前自己这个应用, 而且时间还没有到期
    // 那就直接返回false, 因为已经无法抢占 时间没有过期
    if len(oldLeaderElectionRecord.HolderIdentity) > 0 &&
        le.observedTime.Add(le.config.LeaseDuration).After(now.Time) &&
        !le.IsLeader() {
        klog.V(4).Infof("lock is held by %v and has not yet expired", oldLeaderElectionRecord.HolderIdentity)
        return false
    }
 
    // 3. We're going to try to update. The leaderElectionRecord is set to it's default
    // here. Let's correct it before updating.
    if le.IsLeader() {
        // 如果当前服务就是以前的占有者
        leaderElectionRecord.AcquireTime = oldLeaderElectionRecord.AcquireTime
        leaderElectionRecord.LeaderTransitions = oldLeaderElectionRecord.LeaderTransitions
    } else {
        // 如果当前服务不是以前的占有者 LeaderTransitions加1
        leaderElectionRecord.LeaderTransitions = oldLeaderElectionRecord.LeaderTransitions + 1
    }
 
    // update the lock itself
    // 当前client占有该资源 成为leader
    if err = le.config.Lock.Update(leaderElectionRecord); err != nil {
        klog.Errorf("Failed to update lock: %v", err)
        return false
    }
    le.observedRecord = leaderElectionRecord
    le.observedTime = le.clock.Now()
    return true
}
————————————————
版权声明：本文为CSDN博主「Teingi」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_40449300/article/details/110729620
```

这里需要注意的是当前client不是leader的时候, 如何去判断一个leader是否已经expired了?

* le.observedTime.Add\(le.config.LeaseDuration\).After\(now.Time\) 
* **le.observedTime**: 代表的是获得leader\(截止当前时间为止的最后一次renew\)对象的时间.
* **le.config.LeaseDuration**: 自己\(当前client\)获得leadership需要的等待时间.
* **le.observedTime.Add\(le.config.LeaseDuration\)**: 就是自己\(当前client\)被允许获得leadership的时间.

如果le.observedTime.Add\(le.config.LeaseDuration\).before\(now.Time\)为true的话, 就表明leader过期了. 白话文的意思就是从leader上次续约完, 已经超过le.config.LeaseDuration的时间没有续约了, 所以被认为该leader过期了. 把before换成after就是表明没有过期.

## 四、结语

 leaderelection 主要是利用了k8s API操作的原子性实现了一个分布式锁，在不断的竞争中进行选举。选中为leader的进行才会执行具体的业务代码，这在k8s中非常的常见，而且我们很方便的利用这个包完成组件的编写，从而实现组件的高可用，比如部署为一个多副本的Deployment，当leader的pod退出后会重新启动，可能锁就被其他pod获取继续执行。

## References

* 原文 [【k8s】——leaderelection选举策略实现组件高可用](https://blog.csdn.net/weixin_40449300/article/details/110729620)
* [\[k8s源码分析\]\[client-go\] k8s选举leaderelection \(分布式资源锁实现\)](https://www.jianshu.com/p/6e6f1d97d635)
* [利用 Kubernetes 中的 leaderelection 实现组件高可用](https://www.jianshu.com/p/6e6f1d97d635)

