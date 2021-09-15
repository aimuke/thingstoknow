# Consul原理解析

 发表于 2019-01-04 \|  分类于 [distributed-system ](http://ljchen.net/categories/distributed-system/)\|  阅读时长 ≈ 19 mins

从2016年开始接触微服务的时候就使用consul，当初只知道其特别方便，是一款不错的服务注册与发现工具。至于其部署架构，实现原理都没有深入去了解过，就如同年少读书不求甚解。最近，在着手搞微服务治理，服务治理与发现这块正好选型consul，这才详细的琢磨了下其代码，也对其原理有了一定的认识。下面就听我就徐徐道来……  


## 架构介绍 <a id="&#x67B6;&#x6784;&#x4ECB;&#x7ECD;"></a>

下面是consul官方给出的一张架构图，我们先来理解一下

![](../.gitbook/assets/image%20%2877%29.png)

首先，从架构上，图片被两个datacenter分成了上下两部分；但这两部分又并不是完全隔离的，他们之间通过WAN GOSSIP在Internet上交互报文。因此，我们了解到consul是可以支持多个数据中心之间基于WAN来做同步的。

再看单个datacenter内部，节点被划分为两种颜色，其中红色为server，紫色为client。它们之间通过GRPC通信（主要用于业务数据）。除此之外，server和client之间，还有一条LAN GOSSIP通信，这是用于当LAN内部发生了拓扑变化时，存活的节点们能够及时感知，比如server节点down掉后，client就会触发将对应server节点从可用列表中剥离出去。

当然，server与server之间，client与client之间，client与server之间，在同一个datacenter中的所有consul agent会组成一个LAN网络（当然它们之间也可以按照区域划分segment），当LAN网中有任何角色变动，或者有用户自定义的event产生的时候，其他节点就会感知到，并触发对应的预置操作。

所有的server节点共同组成了一个集群，他们之间运行raft协议，通过共识仲裁选举出leader。所有的业务数据都通过leader写入到集群中做持久化，当有半数以上的节点存储了该数据后，server集群才会返回ACK，从而保障了数据的强一致性。当然，server数量大了之后，也会影响写数据的效率。所有的follower会跟随leader的脚步，保障其有最新的数据副本。

同一个consul agent程序，通过启动的时候指定不同的参数来运行server或client模式。这两种模式下，各自所负责的事务具体如下。

### Server节点 <a id="Server&#x8282;&#x70B9;"></a>

* 参与共识仲裁\(raft\)
* 存储群集状态\(日志存储\)
* 处理查询
* 维护与周边\(LAN/WAN\)各节点关系

### Agent节点 <a id="Agent&#x8282;&#x70B9;"></a>

* 负责通过该节点注册到consul的微服务的健康检查
* 将客户端注册请求以及查询转化为对server的RPC请求
* 维护与周边\(LAN/WAN\)各节点关系

### 服务端口 <a id="&#x670D;&#x52A1;&#x7AEF;&#x53E3;"></a>

| 端口 | 作用 |
| :--- | :--- |
| 8300 | RPC exchanges |
| 8301 | LAN GOSSIP |
| 8302 | WAN GOSSIP |
| 8400 | RPC exchanges by the CLI |
| 8500 | Used for HTTP API and web interface |
| 8600 | Used for DNS server |

## 实现原理 <a id="&#x5B9E;&#x73B0;&#x539F;&#x7406;"></a>

纵观consul的实现，其核心在于两点：

1. 集群内节点间信息的高效同步机制，其保障了拓扑变动以及控制信号的及时传递；
2. server集群内日志存储的强一致性。

它们主要基于以下两个协议来实现：

* 使用gossip协议在集群内传播信息
* 使用raft协议来保障日志的一致性

### Serf <a id="Serf"></a>

serf是hashicorp基于GOSSIP协议来实现的一个用于分布式集群成员管理，失败检测以及编排的工具，当前最新版本为v0.8.1。有兴趣的朋友可以到这个链接具体了解[hashicorp serf](https://www.serf.io/)，下面我来简单介绍一下其功能。

#### 集群管理 <a id="&#x96C6;&#x7FA4;&#x7BA1;&#x7406;"></a>

这台机器上有两个IP地址，一个是172.20.20.10，另一个为172.20.20.10。我准备启动两个serf agent进程，分别绑定到不同的两个IP地址上，各自叫做agent-one和agent-two。

由于它们启动之后，相互之间是不知道彼此的，我通过执行`serf join`来把它们组成一个LAN serf。这样它们就可以彼此检测到彼此，通过查看serf members可以看到所有的节点以及其健康状况。

```bash
$ serf agent -node=agent-one -bind=172.20.20.10

$ serf agent -node=agent-two -bind=172.20.20.11

$ serf join 172.20.20.11

$ serf members
agent-one     172.20.20.10:7946    alive
agent-two     172.20.20.11:7946    alive
```



#### 事件响应 <a id="&#x4E8B;&#x4EF6;&#x54CD;&#x5E94;"></a>

在前面的步骤中，我们将两个serf进程加入到了同一个LAN中，接下来我们将进行一些更加激动人心的实践。接下来，我们创建了一个脚本\(handler.sh\)，大致内容为:当脚本被调用的时候，会打印出一些具体的信息。然后，我们在启动serf agent的时候，通过参数将该脚本传递给serf agent。这样当收该serf节点收到event时，就会调用用户指定的handler（即执行脚本）。

```bash
$ cat handler.sh

#!/bin/bash
echo
echo "New event: ${SERF_EVENT}. Data follows..."
while read line; do
    printf "${line}\n"
done

$ serf agent -log-level=debug -event-handler=handler.sh

```

发送自定义event

```bash
$ serf event hello-there
```

#### Event类型 <a id="Event&#x7C7B;&#x578B;"></a>

serf指定了下面这些类型的event，各自的作用如下所示：

```bash
member-join    One or more members have joined the cluster.
member-leave   One or more members have gracefully left the cluster.
member-failed  One or more members have failed, meaning that they did not properly respond to ping requests.
member-update  One or more members have updated, likely to update the associated tags
member-reap    Serf has removed one or more members from its list of members. This means a failed node exceeded the reconnect_timeout, or a left node reached the tombstone_timeout.
user           A custom user event, covered later in this guide.
query          A query event, covered later in this guide
```

### Raft <a id="Raft"></a>

由于介绍raft协议的文章已经比较多，我这里就不在详述。这里重点分析一下在consul中，raft协议运作的一些实践和日志。

#### 节点状态变更 <a id="&#x8282;&#x70B9;&#x72B6;&#x6001;&#x53D8;&#x66F4;"></a>

1. 在节点数达到bootstrap-expect的数时，开始启用raft选举
2. 在节点数超过bootstrap-expect数时，其他节点为follower
3. 在leader被干掉后，raft如果判断到节点数依然大于等于bootstrap-expect时，重新选举
4. 逐一干掉节点，当节点数少于bootstrap-expect时，raft协议不再选举，将维持先前的状态。

#### Raft选举日志分析 <a id="Raft&#x9009;&#x4E3E;&#x65E5;&#x5FD7;&#x5206;&#x6790;"></a>

```bash
# 选举日志信息 （bootstrap）

==> Starting Consul agent...
bootstrap_expect > 0: expecting 3 servers
==> Consul agent running!
           Version: 'v1.4.0'
           Node ID: 'f217ca95-e83c-9a1f-9e87-3b5c1f5a82a3'
         Node name: '42ddc7aa3bb6'
        Datacenter: 'dc1' (Segment: '<all>')
            Server: true (Bootstrap: false)
       Client Addr: [127.0.0.1] (HTTP: 8500, HTTPS: -1, gRPC: -1, DNS: 8600)
      Cluster Addr: 172.17.0.2 (LAN: 8301, WAN: 8302)
           Encrypt: Gossip: false, TLS-Outgoing: false, TLS-Incoming: false
==> Log data will now stream in as it occurs:
    2018/12/03 07:21:34 [INFO] raft: Initial configuration (index=0): []
    2018/12/03 07:21:34 [INFO] serf: EventMemberJoin: 42ddc7aa3bb6.dc1 172.17.0.2
    2018/12/03 07:21:34 [INFO] serf: EventMemberJoin: 42ddc7aa3bb6 172.17.0.2
    2018/12/03 07:21:34 [INFO] raft: Node at 172.17.0.2:8300 [Follower] entering Follower state (Leader: "")
    2018/12/03 07:21:34 [INFO] consul: Adding LAN server 42ddc7aa3bb6 (Addr: tcp/172.17.0.2:8300) (DC: dc1)
    2018/12/03 07:21:34 [INFO] consul: Handled member-join event for server "42ddc7aa3bb6.dc1" in area "wan"
    2018/12/03 07:21:34 [INFO] agent: Started DNS server 127.0.0.1:8600 (tcp)
    2018/12/03 07:21:34 [INFO] agent: Started DNS server 127.0.0.1:8600 (udp)
    2018/12/03 07:21:34 [INFO] agent: Started HTTP server on 127.0.0.1:8500 (tcp)
    2018/12/03 07:21:34 [INFO] agent: started state syncer
    2018/12/03 07:21:34 [INFO] agent: Retry join LAN is supported for: aliyun aws azure digitalocean gce k8s os packet scaleway softlayer triton vsphere
    2018/12/03 07:21:34 [INFO] agent: Joining LAN cluster...
    2018/12/03 07:21:34 [INFO] agent: (LAN) joining: [172.17.0.2]
    2018/12/03 07:21:34 [INFO] agent: (LAN) joined: 1 Err: <nil>
    2018/12/03 07:21:34 [INFO] agent: Join LAN completed. Synced with 1 initial agents
    # node数量没有达到，无法开始选举
    2018/12/03 07:21:41 [ERR] agent: failed to sync remote state: No cluster leader
    2018/12/03 07:21:43 [WARN] raft: no known peers, aborting election
    2018/12/03 07:21:54 [INFO] serf: EventMemberJoin: 4eb2b75f454a 172.17.0.3
    2018/12/03 07:21:54 [INFO] consul: Adding LAN server 4eb2b75f454a (Addr: tcp/172.17.0.3:8300) (DC: dc1)
    2018/12/03 07:21:54 [INFO] serf: EventMemberJoin: 4eb2b75f454a.dc1 172.17.0.3
    2018/12/03 07:21:54 [INFO] consul: Handled member-join event for server "4eb2b75f454a.dc1" in area "wan"
    2018/12/03 07:21:58 [INFO] serf: EventMemberJoin: b603f61d1449 172.17.0.4
    2018/12/03 07:21:58 [INFO] consul: Adding LAN server b603f61d1449 (Addr: tcp/172.17.0.4:8300) (DC: dc1)
    # node数量达到，开始选举
    2018/12/03 07:21:58 [INFO] consul: Found expected number of peers, attempting bootstrap: 172.17.0.2:8300,172.17.0.3:8300,172.17.0.4:8300
    2018/12/03 07:21:58 [INFO] serf: EventMemberJoin: b603f61d1449.dc1 172.17.0.4
    2018/12/03 07:21:58 [INFO] consul: Handled member-join event for server "b603f61d1449.dc1" in area "wan"
    2018/12/03 07:22:03 [WARN] raft: Heartbeat timeout from "" reached, starting election
    # 状态迁移
    2018/12/03 07:22:03 [INFO] raft: Node at 172.17.0.2:8300 [Candidate] entering Candidate state in term 2
    # 获胜
    2018/12/03 07:22:03 [INFO] raft: Election won. Tally: 2
    2018/12/03 07:22:03 [INFO] raft: Node at 172.17.0.2:8300 [Leader] entering Leader state
    2018/12/03 07:22:03 [INFO] raft: Added peer 5b0b26fb-5e62-c390-0ced-b80e0f3293ef, starting replication
    2018/12/03 07:22:03 [INFO] raft: Added peer 3844affd-9b4e-ad3d-84f3-25fb77806e7c, starting replication
    2018/12/03 07:22:03 [INFO] consul: cluster leadership acquired
    2018/12/03 07:22:03 [INFO] consul: New leader elected: 42ddc7aa3bb6
    2018/12/03 07:22:03 [WARN] raft: AppendEntries to {Voter 3844affd-9b4e-ad3d-84f3-25fb77806e7c 172.17.0.4:8300} rejected, sending older logs (next: 1)
    2018/12/03 07:22:03 [WARN] raft: AppendEntries to {Voter 5b0b26fb-5e62-c390-0ced-b80e0f3293ef 172.17.0.3:8300} rejected, sending older logs (next: 1)
    2018/12/03 07:22:03 [INFO] raft: pipelining replication to peer {Voter 3844affd-9b4e-ad3d-84f3-25fb77806e7c 172.17.0.4:8300}
    2018/12/03 07:22:03 [INFO] raft: pipelining replication to peer {Voter 5b0b26fb-5e62-c390-0ced-b80e0f3293ef 172.17.0.3:8300}
    2018/12/03 07:22:03 [INFO] consul: member '42ddc7aa3bb6' joined, marking health alive
    2018/12/03 07:22:03 [INFO] consul: member '4eb2b75f454a' joined, marking health alive
    2018/12/03 07:22:03 [INFO] consul: member 'b603f61d1449' joined, marking health alive
==> Failed to check for updates: Get https://checkpoint-api.hashicorp.com/v1/check/consul?arch=amd64&os=linux&signature=2ba01aad-86ad-32b1-2cff-dc77537fa0dd&version=1.4.0: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
    2018/12/03 07:22:05 [INFO] agent: Synced node info
    2018/12/03 07:27:25 [INFO] serf: EventMemberJoin: 212149acfdcd 172.17.0.5
    2018/12/03 07:27:25 [INFO] consul: Adding LAN server 212149acfdcd (Addr: tcp/172.17.0.5:8300) (DC: dc1)
    # 添加的新节点为novoter角色，无法参与选举
    2018/12/03 07:27:25 [INFO] raft: Updating configuration with AddNonvoter (45c059ba-bd5c-5a00-f7d6-e490324e7b52, 172.17.0.5:8300) to [{Suffrage:Voter ID:f217ca95-e83c-9a1f-9e87-3b5c1f5a82a3 Address:172.17.0.2:8300} {Suffrage:Voter ID:5b0b26fb-5e62-c390-0ced-b80e0f3293ef Address:172.17.0.3:8300} {Suffrage:Voter ID:3844affd-9b4e-ad3d-84f3-25fb77806e7c Address:172.17.0.4:8300} {Suffrage:Nonvoter ID:45c059ba-bd5c-5a00-f7d6-e490324e7b52 Address:172.17.0.5:8300}]
    2018/12/03 07:27:25 [INFO] serf: EventMemberJoin: 212149acfdcd.dc1 172.17.0.5
    2018/12/03 07:27:25 [INFO] consul: Handled member-join event for server "212149acfdcd.dc1" in area "wan"
    2018/12/03 07:27:25 [INFO] raft: Added peer 45c059ba-bd5c-5a00-f7d6-e490324e7b52, starting replication
    2018/12/03 07:27:25 [WARN] raft: AppendEntries to {Nonvoter 45c059ba-bd5c-5a00-f7d6-e490324e7b52 172.17.0.5:8300} rejected, sending older logs (next: 1)
    2018/12/03 07:27:25 [INFO] consul: member '212149acfdcd' joined, marking health alive
    2018/12/03 07:27:25 [INFO] raft: pipelining replication to peer {Nonvoter 45c059ba-bd5c-5a00-f7d6-e490324e7b52 172.17.0.5:8300}
    2018/12/03 07:27:28 [INFO] serf: EventMemberJoin: edb64d232050 172.17.0.6
    2018/12/03 07:27:28 [INFO] consul: Adding LAN server edb64d232050 (Addr: tcp/172.17.0.6:8300) (DC: dc1)
    # 添加的新节点为novoter角色，无法参与选举
    2018/12/03 07:27:28 [INFO] raft: Updating configuration with AddNonvoter (46ebd85c-5e96-f9bd-81e4-0a82d3b405c7, 172.17.0.6:8300) to [{Suffrage:Voter ID:f217ca95-e83c-9a1f-9e87-3b5c1f5a82a3 Address:172.17.0.2:8300} {Suffrage:Voter ID:5b0b26fb-5e62-c390-0ced-b80e0f3293ef Address:172.17.0.3:8300} {Suffrage:Voter ID:3844affd-9b4e-ad3d-84f3-25fb77806e7c Address:172.17.0.4:8300} {Suffrage:Nonvoter ID:45c059ba-bd5c-5a00-f7d6-e490324e7b52 Address:172.17.0.5:8300} {Suffrage:Nonvoter ID:46ebd85c-5e96-f9bd-81e4-0a82d3b405c7 Address:172.17.0.6:8300}]
    2018/12/03 07:27:28 [INFO] serf: EventMemberJoin: edb64d232050.dc1 172.17.0.6
    2018/12/03 07:27:28 [INFO] consul: Handled member-join event for server "edb64d232050.dc1" in area "wan"
    2018/12/03 07:27:28 [INFO] raft: Added peer 46ebd85c-5e96-f9bd-81e4-0a82d3b405c7, starting replication
    2018/12/03 07:27:28 [INFO] consul: member 'edb64d232050' joined, marking health alive
    2018/12/03 07:27:28 [WARN] raft: AppendEntries to {Nonvoter 46ebd85c-5e96-f9bd-81e4-0a82d3b405c7 172.17.0.6:8300} rejected, sending older logs (next: 1)
    2018/12/03 07:27:28 [INFO] raft: pipelining replication to peer {Nonvoter 46ebd85c-5e96-f9bd-81e4-0a82d3b405c7 172.17.0.6:8300}
    2018/12/03 07:27:43 [INFO] autopilot: Promoting Server (ID: "45c059ba-bd5c-5a00-f7d6-e490324e7b52" Address: "172.17.0.5:8300") to voter
    2018/12/03 07:27:43 [INFO] raft: Updating configuration with AddStaging (45c059ba-bd5c-5a00-f7d6-e490324e7b52, 172.17.0.5:8300) to [{Suffrage:Voter ID:f217ca95-e83c-9a1f-9e87-3b5c1f5a82a3 Address:172.17.0.2:8300} {Suffrage:Voter ID:5b0b26fb-5e62-c390-0ced-b80e0f3293ef Address:172.17.0.3:8300} {Suffrage:Voter ID:3844affd-9b4e-ad3d-84f3-25fb77806e7c Address:172.17.0.4:8300} {Suffrage:Voter ID:45c059ba-bd5c-5a00-f7d6-e490324e7b52 Address:172.17.0.5:8300} {Suffrage:Nonvoter ID:46ebd85c-5e96-f9bd-81e4-0a82d3b405c7 Address:172.17.0.6:8300}]
    2018/12/03 07:27:43 [INFO] autopilot: Promoting Server (ID: "46ebd85c-5e96-f9bd-81e4-0a82d3b405c7" Address: "172.17.0.6:8300") to voter
    2018/12/03 07:27:43 [INFO] raft: Updating configuration with AddStaging (46ebd85c-5e96-f9bd-81e4-0a82d3b405c7, 172.17.0.6:8300) to [{Suffrage:Voter ID:f217ca95-e83c-9a1f-9e87-3b5c1f5a82a3 Address:172.17.0.2:8300} {Suffrage:Voter ID:5b0b26fb-5e62-c390-0ced-b80e0f3293ef Address:172.17.0.3:8300} {Suffrage:Voter ID:3844affd-9b4e-ad3d-84f3-25fb77806e7c Address:172.17.0.4:8300} {Suffrage:Voter ID:45c059ba-bd5c-5a00-f7d6-e490324e7b52 Address:172.17.0.5:8300} {Suffrage:Voter ID:46ebd85c-5e96-f9bd-81e4-0a82d3b405c7 Address:172.17.0.6:8300}]
```



## 源码架构 <a id="&#x6E90;&#x7801;&#x67B6;&#x6784;"></a>

先来看Consul内部是如何做服务注册与发现的流程，下图是consul客户端向agent注册以及发现目标服务的时序图

![](../.gitbook/assets/image%20%2879%29.png)

通过上图，我们大概知道了在consul agent中，功能分为了consul server和consul agent（client）。在前面[架构介绍](http://ljchen.net/2019/01/04/consul%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90/#%E6%9E%B6%E6%9E%84%E4%BB%8B%E7%BB%8D)中我们已经阐述了server和client各自的职责。

consul源码中，server和client都是在一套代码中，通过指定启动参数的形势来运行consul server。这里我们先来重点讲解一下consul client的内部架构。

### Consul Client架 <a id="Consul-Client&#x67B6;&#x6784;"></a>

![](../.gitbook/assets/image%20%2878%29.png)

上图简要描述了consul client中的各重要服务，以及它们之间的关系。

* `lan serf` 主要职责是维护节点之间的关系，当有节点加入或者离开的时候，所有节点都会接收到对应的event，这里的lan serf就是指对这些event做处理的handler的go routine服务。
* `state sync` 在consul启动的时候，会启动该服务，它监听一个channel，当其他服务有向consul server同步配置的需求的时候，就会像channel中写入event信息；然后就会触发该服务向consul server同步配置信息。这里的同步又分为全同步和部分同步，主要是为了降低网路的负担。
* `gRPC router` 这是对连接到consul server的gRPC连接的维护和负载均衡机制。在该服务中心，一方面会基于lan serf对consul server节点的拓扑变更事件来维护server列表，另一方面也会对到存活server的connection做定期的ping来维护连接列表；除此之外，还能够对server连接做客户端负载均衡。
* `local state` 是一个本地的内存数据库，一般执行sync就是从server将数据同步过来保存到该db中；平时做一些配置更改也会对应更新该db。
* `api` consul是提供了HTTP和CLI两种对外访问方式的，这里所谓的API并不是想说接口的细节，而指的是consul所提供对外API对应controller逻辑实现。比如下一节要讲到的服务注册的API，后面都做了什么业务逻辑，这是很重要的一部分，对于复杂的逻辑一般包括了：更新本地local state，启动对应的go routine来做事，使用gRPC向server更新数据，向sync channel发消息从而触发sync等操作。

### 服务注册流程 <a id="&#x670D;&#x52A1;&#x6CE8;&#x518C;&#x6D41;&#x7A0B;"></a>

基于前面一节的介绍，我们大概能够猜测到服务注册大概都需要些什么样的流程，接下来我们就将以下这块的逻辑

![](../.gitbook/assets/image%20%2876%29.png)

上图是其服务注册API的controller中函数调用的一个简化流程。

* 首先`s.agent.AddService`函数要做的就是将接收到的服务信息做一通校验，然后整理成为local state的数据结构之后保存到本地；但是由于它是一个内存数据库，并不能够持久化，于是再将其保存到本地文件中做持久化。
* 干完这些操作之后，如果该服务没有指定healthcheck操作的话，接下来要做的就是将这个服务注册请求同步到consul server，让raft leader将数据真正持久化到server中，这部分我没有在图上体现出来，但是在代码中确实是这样实现的。
* 对于在注册的时候制定了healthcheck内容的服务，需要继续注册healthcheck。由于consul支持的healthcheck类型较多，这里对其所指定类型做了简单的校验，然后就开始干正事了。启动一个goroutine来专门为这个服务执行定期的健康检查操作，可见，如果该consul agent上注册的服务太多的话，势必消耗很多资源，这就要求我们部署方案要做好规划了。
* 当健康检查的结果与先前的结果不一致的时候，会触发对local state的更新，同时，需要局部同步该服务到consul server上的内容。为什么呢？因为服务的健康状态其实是保存到其check字段下的，而非是service的一个一级属性，这块大家可以下去查阅一下代码。另外，每次状态变更都会触发consul agent通过gRPC调用server的`Catalog.Register`来注册服务，我的理解其实是覆盖先前注册关于该服务的信息。

## 操作实践 <a id="&#x64CD;&#x4F5C;&#x5B9E;&#x8DF5;"></a>

介绍consul agent的配置参数，以及各种使用场景下的命令。

### consul agent参数 <a id="consul-agent&#x53C2;&#x6570;"></a>

```bash
-advertise        通知展现地址用来改变我们给集群中的其他节点展现的地址，一般情况下-bind地址就是展现地址
-bootstrap        用来控制一个server是否在bootstrap模式，在一个datacenter中只能有一个server处于bootstrap模式，当一个server处于bootstrap模式时，可以自己选举为raft leader。
-bootstrap-expect 在一个datacenter中期望提供的server节点数目，当该值提供的时候，consul一直等到达到指定sever数目的时候才会引导整个集群，该标记不能和bootstrap公用
-bind             该地址用来在集群内部的通讯，集群内的所有节点到地址都必须是可达的，默认是0.0.0.0
-client           consul绑定在哪个client地址上，这个地址提供HTTP、DNS、RPC等服务，默认是127.0.0.1
-config-file      明确的指定要加载哪个配置文件
-config-dir       配置文件目录，里面所有以.json结尾的文件都会被加载
-data-dir         提供一个目录用来存放agent的状态，所有的agent允许都需要该目录，该目录必须是稳定的，系统重启后都继续存在
-dc               该标记控制agent允许的datacenter的名称，默认是dc1
-encrypt          指定secret key，使consul在通讯时进行加密，key可以通过consul keygen生成，同一个集群中的节点必须使用相同的key
-join             加入一个已经启动的agent的ip地址，可以多次指定多个agent的地址。如果consul不能加入任何指定的地址中，则agent会启动失败，默认agent启动时不会加入任何节点。
-retry-join       和join类似，但是允许你在第一次失败后进行尝试。
-retry-interval   两次join之间的时间间隔，默认是30s
-retry-max        尝试重复join的次数，默认是0，也就是无限次尝试
-log-level        consul agent启动后显示的日志信息级别。默认是info，可选：trace、debug、info、warn、err。
-node             节点在集群中的名称，在一个集群中必须是唯一的，默认是该节点的主机名
-protocol         consul使用的协议版本
-rejoin           使consul忽略先前的离开，在再次启动后仍旧尝试加入集群中。
-server           定义agent运行在server模式，每个集群至少有一个server，建议每个集群的server不要超过5个
-syslog           开启系统日志功能，只在linux/osx上生效
-ui-dir           提供存放web ui资源的路径，该目录必须是可读的
-pid-file         提供一个路径来存放pid文件，可以使用该文件进行SIGINT/SIGHUP(关闭/更新)agent
```

### 常用命令 <a id="&#x5E38;&#x7528;&#x547D;&#x4EE4;"></a>

#### 开发模式 <a id="&#x5F00;&#x53D1;&#x6A21;&#x5F0F;"></a>

最简单，可以用于本地微服务开发的时候，零时做服务注册与发现工具。请注意的是，开发模式下，consul不会做配置的持久化，当consul服务终止时，之前注册的服务和K/V都会随之丢失！

```bash
docker run -d --name=dev-consul -e CONSUL_BIND_INTERFACE=eth0 consul
```

#### server模式 <a id="server&#x6A21;&#x5F0F;"></a>

```bash
docker run -d --net=host -e 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true}' consul agent -server -bind=<external ip> -retry-join=<root agent ip> -bootstrap-expect=<number of server agents>
```

* 启动server

```bash
docker run -d -e 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true}' consul agent -server -retry-join=172.17.0.2 -bootstrap-expect=3
```

#### client模式 <a id="client&#x6A21;&#x5F0F;"></a>

* 启动client

```bash
docker run -d -e 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true}' consul agent -retry-join=172.17.0.2
```

* 暴露dns

```bash
docker run -d --net=host -e 'CONSUL_ALLOW_PRIVILEGED_PORTS=' consul -dns-port=53 -recursor=8.8.8.8
```

* 查询

```bash
docker exec -t dev-consul consul members
dig @localhost -p 8600 consul.service.consul
```

#### 集群部署实践 <a id="&#x96C6;&#x7FA4;&#x90E8;&#x7F72;&#x5B9E;&#x8DF5;"></a>

下面是部署两个server和一个agent的实例

* s1: 10.200.204.104

```bash
./consul agent -server -bootstrap-expect 1 -data-dir /etc/consul/data -node=s1 -bind=10.200.204.104 -ui -rejoin -config-dir=/etc/consul/conf -client 0.0.0.0
```

* s2: 10.200.204.48

```bash
./consul agent -server -bootstrap-expect 1 -data-dir /etc/consul/data -node=s2 -bind=10.200.204.48 -ui -rejoin -config-dir=/etc/consul/conf -client 0.0.0.0 -retry-join=10.200.204.104
```

* c1: 10.200.204.133

```bash
./consul agent -node=c1 -bind=10.200.204.133 -data-dir /etc/consul/data -ui -rejoin -config-dir=/etc/consul/conf -client 0.0.0.0 -retry-join=10.200.204.48
```

#### 查看raft角色 <a id="&#x67E5;&#x770B;raft&#x89D2;&#x8272;"></a>

```bash
# consul operator raft list-peers
Node          ID                                    Address          State     Voter  RaftProtocol
b603f61d1449  3844affd-9b4e-ad3d-84f3-25fb77806e7c  172.17.0.4:8300  follower  true   3
212149acfdcd  45c059ba-bd5c-5a00-f7d6-e490324e7b52  172.17.0.5:8300  follower  true   3
edb64d232050  46ebd85c-5e96-f9bd-81e4-0a82d3b405c7  172.17.0.6:8300  leader    true   3
```

#### 注册服务到consul <a id="&#x6CE8;&#x518C;&#x670D;&#x52A1;&#x5230;consul"></a>

```bash
cat << EOF >> payload.json

{
  "Datacenter": "dc1",
  "ID": "40e4a748-2192-161a-0510-9bf59fe950b5",
  "Node": "foobar",
  "Address": "192.168.10.10",
  "TaggedAddresses": {
    "lan": "192.168.10.10",
    "wan": "10.0.10.10"
  },
  "NodeMeta": {
    "somekey": "somevalue"
  },
  "Service": {
    "ID": "redis1",
    "Service": "redis",
    "Tags": [
      "primary",
      "v1"
    ],
    "Address": "127.0.0.1",
    "Meta": {
        "redis_version": "4.0"
    },
    "Port": 8000
  },
  "Check": {
    "Node": "foobar",
    "CheckID": "service:redis1",
    "Name": "Redis health check",
    "Notes": "Script based health check",
    "Status": "passing",
    "ServiceID": "redis1",
    "Definition": {
      "TCP": "localhost:8888",
      "Interval": "5s",
      "Timeout": "1s",
      "DeregisterCriticalServiceAfter": "30s"
    }
  },
  "SkipNodeUpdate": false
}
EOF
```



```bash
curl --request PUT --data @payload.json http://127.0.0.1:8500/v1/catalog/register
```

一般不推荐使用catalog注册，而是使用agent来注册，注册到agent如下:

```bash
curl -X PUT -H 'application/json' -d '{"ID": "taobao","Name": "taobao","Tags": ["primary","v1"],"Address": "140.205.94.189","Port": 80,"Meta": {"taobao_version": "4.0"},"EnableTagOverride": false,"Check": {"DeregisterCriticalServiceAfter": "90m","HTTP": "http://140.205.94.189:80/","Interval": "10s"},"Weights": {"Passing": 10,"Warning": 1}}' http://127.0.0.1:8500/v1/agent/service/register
```

#### 配置文件注册 <a id="&#x914D;&#x7F6E;&#x6587;&#x4EF6;&#x6CE8;&#x518C;"></a>

直接将以下json文件保存后存放到`--config-dir`目录下，重启consul服务

```bash
{
  "service":{
    "id": "jetty",
    "name": "jetty",
    "address": "14.215.177.38",
    "port": 80,
    "tags": ["dev"],
    "checks": [
        {
            "http": "http://14.215.177.38:80/",
            "interval": "5s"
        }
    ]
  }
}
```



会发现，在哪一个client节点上注册的服务，对应client节点就会负责做healthcheck，也就意味着，这个节点非常重要，如果做不好高可用，所有注册到上面的服务都有被deregisterd的风险。

#### API 注册 <a id="API-&#x6CE8;&#x518C;"></a>

```bash
curl -X PUT -d '{"id": "ljchen","name": "ljchen","address": "14.215.177.38","port": 80,"tags": ["dev"],"checks": [{"http": "http://14.215.177.38:80/","interval": "5s"}]}' http://127.0.0.1:8500/v1/agent/service/register
```

#### 查询consul中的服务 <a id="&#x67E5;&#x8BE2;consul&#x4E2D;&#x7684;&#x670D;&#x52A1;"></a>

```bash
curl http://127.0.0.1:8500/v1/catalog/service/redis?tag=v1
```

#### 删除node节点 <a id="&#x5220;&#x9664;node&#x8282;&#x70B9;"></a>

```bash
curl -X PUT -H 'application/json' -d '{"Datacenter": "dc1","Node": "node-name"}' http://127.0.0.1:8500/v1/catalog/deregister
```

* **本文作者：** ljchen
* **本文链接：** [http://ljchen.net/2019/01/04/consul原理解析/](http://ljchen.net/2019/01/04/consul%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90/)
* **版权声明：** 本博客所有文章除特别声明外，均采用 [CC BY-NC-SA 3.0](https://creativecommons.org/licenses/by-nc-sa/3.0/) 许可协议。转载请注明出处！

[\# distributed-system](http://ljchen.net/tags/distributed-system/) [\# leader-election](http://ljchen.net/tags/leader-election/) [\# consul](http://ljchen.net/tags/consul/)

## References

* [原文 Consul原理解析](http://ljchen.net/2019/01/04/consul%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90/),[ ljchen](http://ljchen.net/)



