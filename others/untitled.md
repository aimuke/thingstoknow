# OpenStack LBaaS之LBaaS

## 1 基础知识

### 1.1 负载均衡

### 1.2 负载均衡器

#### 1.2.1 HAProxy

#### 1.2.2 KeepAlived

#### 1.2.3 Nginx

参考： [http://www.cnblogs.com/sammyliu/p/4656176.html](http://www.cnblogs.com/sammyliu/p/4656176.html)

## 2 LBaaS

### 2.1 架构

![](<../.gitbook/assets/image (36).png>)

### 2.2 LBaaS V1和V2区别

#### 2.2.1 区别

OpenStack中的网络服务通过neutron-lbaas service plugin提供了两种负载均衡器实现方案：\
● BLaaS v1：Juno版本中引入（Liberty版本中弃用）\
● LBaaS v2：Kilo版本中引入\
LBaaS v1和LBaaS v2这2种实现都使用代理。代理处理HAProxy配置和管理HAProxy守护进程。相对于LBaaS v1负载平衡器，LBaaS v2增加了listeners的概念。LBaaS v2允许在一个负载均衡器IPaddress上配置多个listener ports。\
目前，v1和v2负载均衡器之间不存在迁移路径。如果你选择从v1变为v2，需要重新创建所有的负载均衡器和health monitors。

#### 2.2.2 命令

V1:

```
root@controller:~# neutron help | grep lb-
  lb-agent-hosting-pool             Get loadbalancer agent hosting a pool.
  lb-healthmonitor-associate        Create a mapping between a health monitor and a pool.
  lb-healthmonitor-create           Create a health monitor.
  lb-healthmonitor-delete           Delete a given health monitor.
  lb-healthmonitor-disassociate     Remove a mapping from a health monitor to a pool.
  lb-healthmonitor-list             List health monitors that belong to a given tenant.
  lb-healthmonitor-show             Show information of a given health monitor.
  lb-healthmonitor-update           Update a given health monitor.
  lb-member-create                  Create a member.
  lb-member-delete                  Delete a given member.
  lb-member-list                    List members that belong to a given tenant.
  lb-member-show                    Show information of a given member.
  lb-member-update                  Update a given member.
  lb-pool-create                    Create a pool.
  lb-pool-delete                    Delete a given pool.
  lb-pool-list                      List pools that belong to a given tenant.
  lb-pool-list-on-agent             List the pools on a loadbalancer agent.
  lb-pool-show                      Show information of a given pool.
  lb-pool-stats                     Retrieve stats for a given pool.
  lb-pool-update                    Update a given pool.
  lb-vip-create                     Create a vip.
  lb-vip-delete                     Delete a given vip.
  lb-vip-list                       List vips that belong to a given tenant.
  lb-vip-show                       Show information of a given vip.
  lb-vip-update                     Update a given vip.
```

V2:

```
root@controller:~# neutron help | grep lbaas-
  lbaas-agent-hosting-loadbalancer  Get lbaas v2 agent hosting a loadbalancer.
  lbaas-healthmonitor-create        LBaaS v2 Create a healthmonitor.
  lbaas-healthmonitor-delete        LBaaS v2 Delete a given healthmonitor.
  lbaas-healthmonitor-list          LBaaS v2 List healthmonitors that belong to a given tenant.
  lbaas-healthmonitor-show          LBaaS v2 Show information of a given healthmonitor.
  lbaas-healthmonitor-update        LBaaS v2 Update a given healthmonitor.
  lbaas-listener-create             LBaaS v2 Create a listener.
  lbaas-listener-delete             LBaaS v2 Delete a given listener.
  lbaas-listener-list               LBaaS v2 List listeners that belong to a given tenant.
  lbaas-listener-show               LBaaS v2 Show information of a given listener.
  lbaas-listener-update             LBaaS v2 Update a given listener.
  lbaas-loadbalancer-create         LBaaS v2 Create a loadbalancer.
  lbaas-loadbalancer-delete         LBaaS v2 Delete a given loadbalancer.
  lbaas-loadbalancer-list           LBaaS v2 List loadbalancers that belong to a given tenant.
  lbaas-loadbalancer-list-on-agent  List the loadbalancers on a loadbalancer v2 agent.
  lbaas-loadbalancer-show           LBaaS v2 Show information of a given loadbalancer.
  lbaas-loadbalancer-update         LBaaS v2 Update a given loadbalancer.
  lbaas-member-create               LBaaS v2 Create a member.
  lbaas-member-delete               LBaaS v2 Delete a given member.
  lbaas-member-list                 LBaaS v2 List members that belong to a given pool.
  lbaas-member-show                 LBaaS v2 Show information of a given member.
  lbaas-member-update               LBaaS v2 Update a given member.
  lbaas-pool-create                 LBaaS v2 Create a pool.
  lbaas-pool-delete                 LBaaS v2 Delete a given pool.
  lbaas-pool-list                   LBaaS v2 List pools that belong to a given tenant.
  lbaas-pool-show                   LBaaS v2 Show information of a given pool.
  lbaas-pool-update                 LBaaS v2 Update a given pool.
```

#### 2.2.3 LBaaS V1概念

To use OpenStack LBaaS APIs effectively, you should understand several key concepts:

**VIP**

A VIP is the primary load balancing configuration object that specifies the virtual IP address and port on which client traffic is received, as well as other details such as the load balancing method to be use, protocol, etc. This entity is sometimes known in LB products under the name of a "virtual server", a "vserver" or a "listener".

**Pool**

A load balancing pool is a logical set of devices, such as web servers, that you group together to receive and process traffic. The loadbalancing function chooses a member of the pool according to the configured load balancing method to handle the new requests or connections received on the VIP address. There is only one pool for a VIP.

**Pool Member**

A pool member represents the application running on backend server.

**Health Monitoring**

A health monitor is used to determine whether or not back-end members of the VIP's pool are usable for processing a request. A pool can have several health monitors associated with it. There are different types of health monitors supported by the [OpenStack](https://wiki.openstack.org/wiki/OpenStack) LBaaS service:

* PING: used to ping the members using ICMP.
* TCP: used to connect to the members using TCP.
* HTTP: used to send an HTTP request to the member.
* HTTPS: used to send a secure HTTP request to the member.

**Session Persistence**

Session persistence is a feature of the load balancing service. It attempts to force connections or requests in the same session to be processed by the same member as long as it is ative. The [OpenStack](https://wiki.openstack.org/wiki/OpenStack) LBaaS service supports three types of persistence:

* SOURCE\_IP: With this persistence mode, all connections originating from the same source IP address, will be handled by the same member of the pool.
* HTTP\_COOKIE: With this persistence mode, the loadbalancer will create a cookie on the first request from a client. Subsequent requests containing the same cookie value will be handled by the same member of the pool.
* APP\_COOKIE: With this persistence mode, the loadbalancer will rely on a cookie established by the backend application. All requests carrying the same cookie value will be handled by the same member of the pool.

**Connection Limits**

To control incoming traffic on the VIP address as well as traffic for a specific member of a pool, you can set a connection limit beyond which the load balancing function will refuse client requests or connections. This can be used to thwart DoS attacks and to allow each member to continue to work within its limits.

For HTTP and HTTPS protocols, since several HTTP requests can be multiplexed on the same TCP connection, the connection limit value is interpreted as the maximum number of requests allowed.

![](<../.gitbook/assets/image (1).png>)

#### 2.2.4 LBaaS V2概念

![](<../.gitbook/assets/image (94).png>)

负载均衡器 :负载均衡器占用Neutron网络端口，并具有从子网分配的IP地址。 \
侦听器 :负载平衡器可以侦听多个端口上的请求。 这些端口中的每一个都由侦听器指定。 \
池 :池包含通过负载均衡器提供内容的成员的列表。 \
成员 :成员是为负载均衡器后面的流量提供服务的服务器。 每个成员由用于提供流量的IP地址和端口指定。 \
健康监视器 :成员可能不时离线，健康监视器将流量从没有正确响应的成员转移。 运行状况监视器与池相关联。\


参考：[http://blog.csdn.net/zhaihaifei/article/details/39963163](http://blog.csdn.net/zhaihaifei/article/details/39963163)

## 3 安装配置基于haproxy的负载均衡服务(LBaaS)

### 3.1 安装配置LBaaS V1

安装环境是一个包扩controller节点, network节点和computer节点的标准Openstack环境。\


#### 3.1.1 在network节点安装agent

```
apt-get install neutron-lbaas-agent
```

安装过程：

```
root@network:~# ls /etc/neutron/
api-paste.ini   dnsmasq-neutron.conf  l3_agent.ini        neutron.conf  policy.d     rootwrap.conf
dhcp_agent.ini  fwaas_driver.ini      metadata_agent.ini  plugins       policy.json  rootwrap.d

root@network:~# apt-get install neutron-lbaas-agent
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following extra packages will be installed:
  haproxy neutron-lbaas-common python-barbicanclient python-neutron-lbaas
  python-pyasn1-modules
Suggested packages:
  vim-haproxy haproxy-doc
The following NEW packages will be installed:
  haproxy neutron-lbaas-agent neutron-lbaas-common python-barbicanclient
  python-neutron-lbaas python-pyasn1-modules
0 upgraded, 6 newly installed, 0 to remove and 64 not upgraded.
Need to get 551 kB/922 kB of archives.
After this operation, 4,987 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
...
root@network:~# ps -ef | grep lbaas
neutron  16129     1 30 16:16 ?        00:00:00 /usr/bin/python /usr/bin/neutron-lbaas-agent --config-file=/etc/neutron/neutron.conf --config-file=/etc/neutron/lbaas_agent.ini --log-file=/var/log/neutron/neutron-lbaas-agent.log
root     16138 12914  0 16:16 pts/1    00:00:00 grep --color=auto lbaas
root@network:~# ps -ef | grep haproxy
root      2534 12914  0 16:25 pts/1    00:00:00 grep --color=auto haproxy
haproxy  13822     1  0 16:15 ?        00:00:00 /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -D -p /var/run/haproxy.pid
root@network:~# ls /etc/neutron/
api-paste.ini   dnsmasq-neutron.conf  l3_agent.ini     metadata_agent.ini  neutron_lbaas.conf  policy.d     rootwrap.conf  services_lbaas.conf
dhcp_agent.ini  fwaas_driver.ini      lbaas_agent.ini  neutron.conf        plugins             policy.json  rootwrap.d

多出3个文件：lbaas_agent.ini、neutron_lbaas.conf、services_lbaas.conf
```

#### 3.1.2 配置

Lbaas主要分两部分plugin、providers和agent，\


1\. 在controller节点的配置

1.1)配置服务插件plugin，修改/etc/neutron/neutron.conf，添加如下内容：

```
[DEFAULT]  
service_plugins = lbaas 
```

注意：如果已使用使用service\_plugins，需要将lbass也加入，如下：

```
[DEFAULT]
service_plugins = router,lbaas
```

1.2).配置service provider，修改/etc/neutron/neutron\_lbaas.conf，添加如下内容：

```
[service_providers]
# Must be in form:
# service_provider=<service_type>:<name>:<driver>[:default]
# List of allowed service types includes LOADBALANCER
# Combination of <service type> and <name> must be unique; <driver> must also be unique
# This is multiline option
# service_provider=LOADBALANCER:name:lbaas_plugin_driver_path:default
#service_provider=LOADBALANCER:Haproxy:neutron_lbaas.services.loadbalancer.drivers.haproxy.plugin_driver.HaproxyOnHostPluginDriver:default
# service_provider = LOADBALANCER:radware:neutron_lbaas.services.loadbalancer.drivers.radware.driver.LoadBalancerDriver:default
# service_provider = LOADBALANCERV2:radwarev2:neutron_lbaas.drivers.radware.v2_driver.RadwareLBaaSV2Driver:default
# service_provider=LOADBALANCER:NetScaler:neutron_lbaas.services.loadbalancer.drivers.netscaler.netscaler_driver.NetScalerPluginDriver
# service_provider=LOADBALANCER:Embrane:neutron_lbaas.services.loadbalancer.drivers.embrane.driver.EmbraneLbaas:default
# service_provider = LOADBALANCER:A10Networks:neutron_lbaas.services.loadbalancer.drivers.a10networks.driver_v1.ThunderDriver:default
# service_provider = LOADBALANCER:VMWareEdge:neutron_lbaas.services.loadbalancer.drivers.vmware.edge_driver.EdgeLoadbalancerDriver:default

# LBaaS v2 drivers
# service_provider = LOADBALANCERV2:Octavia:neutron_lbaas.drivers.octavia.driver.OctaviaDriver:default
# service_provider = LOADBALANCERV2:LoggingNoop:neutron_lbaas.drivers.logging_noop.driver.LoggingNoopLoadBalancerDriver:default
# service_provider=LOADBALANCERV2:Haproxy:neutron_lbaas.drivers.haproxy.plugin_driver.HaproxyOnHostPluginDriver:default
# service_provider = LOADBALANCERV2:A10Networks:neutron_lbaas.drivers.a10networks.driver_v2.ThunderDriver:default
# service_provider = LOADBALANCERV2:brocade:neutron_lbaas.drivers.brocade.driver_v2.BrocadeLoadBalancerDriver:default
# service_provider = LOADBALANCERV2:kemptechnologies:neutron_lbaas.drivers.kemptechnologies.driver_v2.KempLoadMasterDriver:default
#service_provider=LOADBALANCERV2:Haproxy:neutron_lbaas.drivers.haproxy.plugin_driver.HaproxyOnHostPluginDriver:default
service_provider=LOADBALANCER:Haproxy:neutron_lbaas.services.loadbalancer.drivers.haproxy.plugin_driver.HaproxyOnHostPluginDriver:default
```

1.3) 启动neutron-server服务

```
root@controller:~# service neutron-server restart
```

2\. 在network节点的配置

2.1) 配置device\_driver，修改/etc/neutron/lbaas\_agent.ini，添加如下内容。注意在liberty版本里的device\_driver必须是"neutron\_lbaas.services.loadbalancer.drivers.haproxy.namespace\_driver.HaproxyNSDriver"，因为旧的"neutron.services.loadbalancer.drivers.haproxy.namespace\_driver.HaproxyNSDriver"已经被移除了。

2.2) 配置interface\_driver

Enable the Open vSwitch LBaaS driver: interface\_driver = neutron.agent.linux.interface.OVSInterfaceDriver\
enable the Linux Bridge LBaaS driver: interface\_driver = neutron.agent.linux.interface.BridgeInterfaceDriver\


\
配置后：

```
[DEFAULT]
# Show debugging output in log (sets DEBUG log level output).
# debug = False

# The LBaaS agent will resync its state with Neutron to recover from any
# transient notification or rpc errors. The interval is number of
# seconds between attempts.
# periodic_interval = 10

# LBaas requires an interface driver be set. Choose the one that best
# matches your plugin.
# interface_driver =

# Example of interface_driver option for OVS based plugins (OVS, Ryu, NEC, NVP,
# BigSwitch/Floodlight)
# interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver

# Use veth for an OVS interface or not.
# Support kernels with limited namespace support
# (e.g. RHEL 6.5) so long as ovs_use_veth is set to True.
# ovs_use_veth = False

# Example of interface_driver option for LinuxBridge
# interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver

# The agent requires drivers to manage the loadbalancer.  HAProxy is the opensource version.
# Multiple device drivers reflecting different service providers could be specified:
# device_driver = path.to.provider1.driver.Driver
# device_driver = path.to.provider2.driver.Driver
# Default is:
# device_driver = neutron_lbaas.services.loadbalancer.drivers.haproxy.namespace_driver.HaproxyNSDriver
device_driver = neutron_lbaas.services.loadbalancer.drivers.haproxy.namespace_driver.HaproxyNSDriver

[haproxy]
# Location to store config and state files
# loadbalancer_state_path = $state_path/lbaas

# The user group
# user_group = nogroup
user_group = haproxy
```

2.3) 启动neutron-lbaas-agent服务：

```
root@network:~# service neutron-lbaas-agent restart
```

3 Enable load balancing in the Project section of the dashboard.\
Change the enable\_lb option to True in the/etc/openstack-dashboard/local\_settings file:\


```
OPENSTACK_NEUTRON_NETWORK = {
'enable_lb': True,
...
}
```

Apply the settings by restarting the httpd service. You can now view the Load Balancer management options in the Project view in the dashboard.

2.4) 检查neutron-lbaas-agent服务：

```
root@controller:~# neutron agent-list
+--------------------------------------+--------------------+---------+-------+----------------+---------------------------+
| id                                   | agent_type         | host    | alive | admin_state_up | binary                    |
+--------------------------------------+--------------------+---------+-------+----------------+---------------------------+
| 0a631df1-0d60-4b51-bd63-e13fde4ac169 | Metadata agent     | network | :-)   | True           | neutron-metadata-agent    |
| 5d15c5ed-b6de-4214-9f04-7cf37e23a360 | Linux bridge agent | network | :-)   | True           | neutron-linuxbridge-agent |
| 917087f6-effa-4ce3-b641-c5acdbbe293c | L3 agent           | network | :-)   | True           | neutron-l3-agent          |
| 9b7e6fff-0494-4e9c-ad24-dec71960ef79 | Loadbalancer agent | network | :-)   | True           | neutron-lbaas-agent       |
| b3095a2c-9d2b-4180-8fca-c6d1590a500e | Linux bridge agent | compute | :-)   | True           | neutron-linuxbridge-agent |
| dde30b8d-c0b5-417f-b7b6-b1345fa43889 | DHCP agent         | network | :-)   | True           | neutron-dhcp-agent        |
+--------------------------------------+--------------------+---------+-------+----------------+---------------------------+
```

#### 3.1.3 操作

3.1.3.1 命令行

This list shows example neutron commands that enable you to complete basic LBaaS operations:\
• Creates a load balancer pool by using specific provider.\
\--provider is an optional argument. If not used, the pool is created with default provider for LBaaS service. You should configure the default provider in the\
\[service\_providers] section of neutron.conf file. If no default provider is specified for LBaaS, the --provider option is required for pool creation.\


```
$ neutron lb-pool-create --lb-method ROUND_ROBIN --name mypool --protocol HTTP --subnet-id SUBNET_UUID --provider PROVIDER_NAME
```

• Associates two web servers with pool.\


```
$ neutron lb-member-create --address WEBSERVER1_IP --protocol-port 80 mypool
```

```
$ neutron lb-member-create --address WEBSERVER2_IP --protocol-port 80 mypool
```

• Creates a health monitor that checks to make sure our instances are still running on the specified protocol-port.\


```
$ neutron lb-healthmonitor-create --delay 3 --type HTTP --max-retries 3 --timeout 3
```

• Associates a health monitor with pool.\


```
$ neutron lb-healthmonitor-associate HEALTHMONITOR_UUID mypool
```

• Creates a virtual IP (VIP) address that, when accessed through the load balancer, directs the requests to one of the pool members.\


```
$ neutron lb-vip-create --name myvip --protocol-port 80 --protocol HTTP --subnet-id SUBNET_UUID mypool
```

3.1.3.2 界面操作

参考：[http://blog.csdn.net/CloudMan6/article/details/53461562](http://blog.csdn.net/CloudMan6/article/details/53461562)

### 3.2 安装配置LBaaS V2

安装环境是一个包扩controller节点, network节点和computer节点的标准Openstack环境。\


#### 3.2.1 在controller和network节点安装agent

apt-get install neutron-lbaasv2-agent\
安装过程：\


```
root@network:~# ls /etc/neutron/
api-paste.ini   dnsmasq-neutron.conf  l3_agent.ini        neutron.conf  policy.d     rootwrap.conf
dhcp_agent.ini  fwaas_driver.ini      metadata_agent.ini  plugins       policy.json  rootwrap.d


root@network:~# apt-get install neutron-lbaasv2-agent
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following extra packages will be installed:
  haproxy neutron-lbaas-common python-barbicanclient python-neutron-lbaas
  python-pyasn1-modules
Suggested packages:
  vim-haproxy haproxy-doc
The following NEW packages will be installed:
  haproxy neutron-lbaas-common neutron-lbaasv2-agent python-barbicanclient
  python-neutron-lbaas python-pyasn1-modules
0 upgraded, 6 newly installed, 0 to remove and 64 not upgraded.
Need to get 0 B/922 kB of archives.
After this operation, 4,987 kB of additional disk space will be used.
Do you want to continue? [Y/n] y 
...
root@network:~# ps -ef | grep lbaas
neutron  16129     1 30 16:16 ?        00:00:00 /usr/bin/python /usr/bin/neutron-lbaasv2-agent --config-file=/etc/neutron/neutron.conf --config-file=/etc/neutron/lbaas_agent.ini --log-file=/var/log/neutron/neutron-lbaasv2-agent.log
root     16138 12914  0 16:16 pts/1    00:00:00 grep --color=auto lbaas
root@network:~# ps -ef | grep haproxy
root      2534 12914  0 16:25 pts/1    00:00:00 grep --color=auto haproxy
haproxy  13822     1  0 16:15 ?        00:00:00 /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -D -p /var/run/haproxy.pid
root@network:~# ls /etc/neutron/
api-paste.ini   dnsmasq-neutron.conf  l3_agent.ini     metadata_agent.ini  neutron_lbaas.conf  policy.d     rootwrap.conf  services_lbaas.conf
dhcp_agent.ini  fwaas_driver.ini      lbaas_agent.ini  neutron.conf        plugins             policy.json  rootwrap.d

多出3个文件：lbaas_agent.ini、neutron_lbaas.conf、services_lbaas.conf
```

\
如果不在controller节点安装，会找不到service\_plugins出现错误：\


```
root@controller:~# neutron lbaas-loadbalancer-list
Unable to establish connection to http://controller:9696/v2.0/lbaas/loadbalancers.json
root@controller:~# tailf /var/log/neutron/neutron-server.log
RuntimeError: No 'neutron.service_plugins' driver found
ImportError: No module named neutron_lbaas.services.loadbalancer.plugin
```

#### 3.2.2 配置

Lbaas主要分两部分plugin、providers和agent，\


**3.2.2.1. 在controller节点的配置**

1.1)配置服务插件plugin，修改/etc/neutron/neutron.conf，添加如下内容：\


```
[DEFAULT]  
service_plugins = neutron_lbaas.services.loadbalancer.plugin.LoadBalancerPluginv2 
```

\
注意：如果已使用使用service\_plugins，需要将lbass也加入，如下：\


```
[DEFAULT]
service_plugins = router,neutron_lbaas.services.loadbalancer.plugin.LoadBalancerPluginv2
```

\
1.2).配置service provider，修改/etc/neutron/neutron\_lbaas.conf，添加如下内容：\


```
[service_providers]
# Must be in form:
# service_provider=<service_type>:<name>:<driver>[:default]
# List of allowed service types includes LOADBALANCER
# Combination of <service type> and <name> must be unique; <driver> must also be unique
# This is multiline option
# service_provider=LOADBALANCER:name:lbaas_plugin_driver_path:default
#service_provider=LOADBALANCER:Haproxy:neutron_lbaas.services.loadbalancer.drivers.haproxy.plugin_driver.HaproxyOnHostPluginDriver:default
# service_provider = LOADBALANCER:radware:neutron_lbaas.services.loadbalancer.drivers.radware.driver.LoadBalancerDriver:default
# service_provider = LOADBALANCERV2:radwarev2:neutron_lbaas.drivers.radware.v2_driver.RadwareLBaaSV2Driver:default
# service_provider=LOADBALANCER:NetScaler:neutron_lbaas.services.loadbalancer.drivers.netscaler.netscaler_driver.NetScalerPluginDriver
# service_provider=LOADBALANCER:Embrane:neutron_lbaas.services.loadbalancer.drivers.embrane.driver.EmbraneLbaas:default
# service_provider = LOADBALANCER:A10Networks:neutron_lbaas.services.loadbalancer.drivers.a10networks.driver_v1.ThunderDriver:default
# service_provider = LOADBALANCER:VMWareEdge:neutron_lbaas.services.loadbalancer.drivers.vmware.edge_driver.EdgeLoadbalancerDriver:default

# LBaaS v2 drivers
# service_provider = LOADBALANCERV2:Octavia:neutron_lbaas.drivers.octavia.driver.OctaviaDriver:default
# service_provider = LOADBALANCERV2:LoggingNoop:neutron_lbaas.drivers.logging_noop.driver.LoggingNoopLoadBalancerDriver:default
# service_provider=LOADBALANCERV2:Haproxy:neutron_lbaas.drivers.haproxy.plugin_driver.HaproxyOnHostPluginDriver:default
# service_provider = LOADBALANCERV2:A10Networks:neutron_lbaas.drivers.a10networks.driver_v2.ThunderDriver:default
# service_provider = LOADBALANCERV2:brocade:neutron_lbaas.drivers.brocade.driver_v2.BrocadeLoadBalancerDriver:default
# service_provider = LOADBALANCERV2:kemptechnologies:neutron_lbaas.drivers.kemptechnologies.driver_v2.KempLoadMasterDriver:default
service_provider=LOADBALANCERV2:Haproxy:neutron_lbaas.drivers.haproxy.plugin_driver.HaproxyOnHostPluginDriver:default
```

\
1.3) 启动neutron-server服务\
root@controller:\~# service neutron-server restart\


**3.2.2.2. 在network节点的配置**

2.1) 不用配置device\_driver。如果配置，在创建loadbalancer时，后出错：\


```
root@network:~# vi /var/log/neutron/neutron-lbaasv2-agent.log
AttributeError: 'HaproxyNSDriver' object has no attribute 'loadbalancer'
```

\
2.2) 配置interface\_driver，修改/etc/neutron/lbaas\_agent.ini\
Enable the Open vSwitch LBaaS driver: interface\_driver = neutron.agent.linux.interface.OVSInterfaceDriver\
enable the Linux Bridge LBaaS driver: interface\_driver = neutron.agent.linux.interface.BridgeInterfaceDriver\
配置后：\


```
[DEFAULT]
# Show debugging output in log (sets DEBUG log level output).
# debug = False

# The LBaaS agent will resync its state with Neutron to recover from any
# transient notification or rpc errors. The interval is number of
# seconds between attempts.
# periodic_interval = 10

# LBaas requires an interface driver be set. Choose the one that best
# matches your plugin.
# interface_driver =

# Example of interface_driver option for OVS based plugins (OVS, Ryu, NEC, NVP,
# BigSwitch/Floodlight)
# interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver

# Use veth for an OVS interface or not.
# Support kernels with limited namespace support
# (e.g. RHEL 6.5) so long as ovs_use_veth is set to True.
# ovs_use_veth = False

# Example of interface_driver option for LinuxBridge
# interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver

# The agent requires drivers to manage the loadbalancer.  HAProxy is the opensource version.
# Multiple device drivers reflecting different service providers could be specified:
# device_driver = path.to.provider1.driver.Driver
# device_driver = path.to.provider2.driver.Driver
# Default is:
# device_driver = neutron_lbaas.services.loadbalancer.drivers.haproxy.namespace_driver.HaproxyNSDriver

[haproxy]
# Location to store config and state files
# loadbalancer_state_path = $state_path/lbaas

# The user group
# user_group = nogroup
user_group = haproxy
```

\
2.3) 在控制节点，运行neutron-lbaas数据库迁移：\
neutron-db-manage --subproject neutron-lbaas upgrade head\
如果不迁移，会出错：\


```
2017-07-10 11:21:50.162 15238 ERROR neutron.service DBError: (pymysql.err.InternalError) (1054, u"Unknown column 'lbaas_loadbalancers.operating_status' in 'field list'") 
```

如果您已部署LBaaS v1，现在停止LBaaS v1代理。 v1和v2代理无法同时运行。 \
2.4) 启动neutron-lbaasv2-agent服务：\


```
root@network:~# service neutron-lbaasv2-agent restart
neutron-lbaasv2-agent stop/waiting
neutron-lbaasv2-agent start/running, process 24265
```

2.5) 重新启动网络服务以激活新配置。 \


```
root@controller:~# service neutron-server restart
neutron-server stop/waiting
neutron-server start/running, process 21566
```

2.6) 检查neutron-lbaasv2-agent服务：\


```
root@controller:~# neutron agent-list
+--------------------------------------+----------------------+---------+-------+----------------+---------------------------+
| id                                   | agent_type           | host    | alive | admin_state_up | binary                    |
+--------------------------------------+----------------------+---------+-------+----------------+---------------------------+
| 0a631df1-0d60-4b51-bd63-e13fde4ac169 | Metadata agent       | network | :-)   | True           | neutron-metadata-agent    |
| 2511844d-4f80-4129-baaa-ae0086e1f079 | Loadbalancerv2 agent | network | :-)   | True           | neutron-lbaasv2-agent     |
| 5d15c5ed-b6de-4214-9f04-7cf37e23a360 | Linux bridge agent   | network | :-)   | True           | neutron-linuxbridge-agent |
| 917087f6-effa-4ce3-b641-c5acdbbe293c | L3 agent             | network | :-)   | True           | neutron-l3-agent          |
| b3095a2c-9d2b-4180-8fca-c6d1590a500e | Linux bridge agent   | compute | :-)   | True           | neutron-linuxbridge-agent |
| dde30b8d-c0b5-417f-b7b6-b1345fa43889 | DHCP agent           | network | :-)   | True           | neutron-dhcp-agent        |
+--------------------------------------+----------------------+---------+-------+----------------+---------------------------+
```

**3.2.2.3 把LBaaS的模块加入仪表板**

用于管理LBaaS v2的仪表板面板可从Mitaka发行版开始提供。 在我实验的Liberty版本中安装失败。\
1.克隆neutron-lbaas-dashboard存储库，并查看与安装的Dashboard版本相匹配的发行版分支：\


```
$ git clone https://git.openstack.org/openstack/neutron-lbaas-dashboard
$ cd neutron-lbaas-dashboard
$ git checkout OPENSTACK_RELEASE
```

2.安装仪表板面板插件：\
$ python setup.py install\
3.将\_1481\_project\_ng\_loadbalancersv2\_panel.py文件从neutron-lbaas-dashboard / enabled目录复制到Dashboard Openstack\_dashboard / local / enabled目录中。 \
此步骤可确保在插件枚举其所有可用面板时，Dashboard可以找到该插件。 \
4.通过在OPENSTACK\_NEUTRON\_NETWORK字典中编辑local\_settings.py文件并将enable\_lb设置为True，在Dashboard中启用插件。 \
5.如果将Dashboard配置为压缩静态文件以获得更好的性能（通常通过local\_settings.py中的COMPRESS\_OFFLINE设置），请再次优化静态文件：\


```
$ ./manage.py collectstatic
$ ./manage.py compress
```

6.重新启动Apache以激活新面板：\


```
$ sudo service apache2 restart
```

要查找面板，请单击仪表板中的项目，然后单击网络下拉菜单，并选择负载平衡器。\


#### 3.2.3 命令行操作

**3.2.3.1 建立一个LBaaS v2 负载均衡器**

1.首先在网络上创建负载均衡器。在此示例中，专用网络private是具有两个Web服务器实例aaa和bbb的隔离网络：\


```
root@controller:~# . /home/stack/demo-openrc.sh 
root@controller:~# neutron subnet-list
+--------------------------------------+---------+----------------+----------------------------------------------------+
| id                                   | name    | cidr           | allocation_pools                                   |
+--------------------------------------+---------+----------------+----------------------------------------------------+
| f0e1f744-21e0-42dc-9958-83c4294894d1 | public  | 192.168.4.0/24 | {"start": "192.168.4.140", "end": "192.168.4.149"} |
| 6b9c6742-f965-4b62-899b-6e60da743e66 | private | 10.0.0.0/24    | {"start": "10.0.0.2", "end": "10.0.0.254"}         |
+--------------------------------------+---------+----------------+----------------------------------------------------+
root@controller:~# 
root@controller:~# neutron lbaas-loadbalancer-create --name lber --vip-address 10.0.0.100 private
Created a new loadbalancer:
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| admin_state_up      | True                                 |
| description         |                                      |
| id                  | 628b6622-154b-4342-8be1-cd645dbb601e |
| listeners           |                                      |
| name                | lber                                 |
| operating_status    | OFFLINE                              |
| provider            | haproxy                              |
| provisioning_status | PENDING_CREATE                       |
| tenant_id           | 0cac10bf2056482cbafde6f696a58f40     |
| vip_address         | 10.0.0.100                           |
| vip_port_id         | 6b5dfa29-03bd-4af7-b41e-5c2de2360304 |
| vip_subnet_id       | 6b9c6742-f965-4b62-899b-6e60da743e66 |
+---------------------+--------------------------------------+
```

\
2.您可以使用neutron lbaas-loadbalancer-show命令查看负载均衡器状态和IP地址：\


```
root@controller:~# neutron lbaas-loadbalancer-show lber
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| admin_state_up      | True                                 |
| description         |                                      |
| id                  | 628b6622-154b-4342-8be1-cd645dbb601e |
| listeners           |                                      |
| name                | lber                                 |
| operating_status    | ONLINE                               |
| provider            | haproxy                              |
| provisioning_status | ACTIVE                               |
| tenant_id           | 0cac10bf2056482cbafde6f696a58f40     |
| vip_address         | 10.0.0.100                           |
| vip_port_id         | 6b5dfa29-03bd-4af7-b41e-5c2de2360304 |
| vip_subnet_id       | 6b9c6742-f965-4b62-899b-6e60da743e66 |
+---------------------+--------------------------------------+
```

\
3.更新安全组以允许流量到达新的负载平衡器。 创建新的安全组以及入口规则，以允许流量进入新的负载平衡器。 负载平衡器的neutron端口在上面显示为vip\_port\_id。 \
创建安全组和规则以允许TCP端口80，TCP端口443和所有ICMP流量：\


```
root@controller:~# neutron security-group-create lbaas
Created a new security_group:
+----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field                | Value                                                                                                                                                                                                                                                                                                                         |
+----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| description          |                                                                                                                                                                                                                                                                                                                               |
| id                   | 34bc14b0-fc25-48ca-92c4-2b10c1d940a6                                                                                                                                                                                                                                                                                          |
| name                 | lbaas                                                                                                                                                                                                                                                                                                                         |
| security_group_rules | {"remote_group_id": null, "direction": "egress", "remote_ip_prefix": null, "protocol": null, "tenant_id": "0cac10bf2056482cbafde6f696a58f40", "port_range_max": null, "security_group_id": "34bc14b0-fc25-48ca-92c4-2b10c1d940a6", "port_range_min": null, "ethertype": "IPv4", "id": "d082b8f9-5e5e-4ded-8d9f-3dcebdcf6df4"} |
|                      | {"remote_group_id": null, "direction": "egress", "remote_ip_prefix": null, "protocol": null, "tenant_id": "0cac10bf2056482cbafde6f696a58f40", "port_range_max": null, "security_group_id": "34bc14b0-fc25-48ca-92c4-2b10c1d940a6", "port_range_min": null, "ethertype": "IPv6", "id": "bcca8e53-df69-49ad-bbdb-2752e4e53a64"} |
| tenant_id            | 0cac10bf2056482cbafde6f696a58f40                                                                                                                                                                                                                                                                                              |
+----------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
root@controller:~# neutron security-group-rule-create --direction ingress --protocol tcp --port-range-min 80 --port-range-max 80 --remote-ip-prefix 0.0.0.0/0 lbaas
Created a new security_group_rule:
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| direction         | ingress                              |
| ethertype         | IPv4                                 |
| id                | 78b2aaf9-645a-4c35-aecb-8dcd050a5f01 |
| port_range_max    | 80                                   |
| port_range_min    | 80                                   |
| protocol          | tcp                                  |
| remote_group_id   |                                      |
| remote_ip_prefix  | 0.0.0.0/0                            |
| security_group_id | 34bc14b0-fc25-48ca-92c4-2b10c1d940a6 |
| tenant_id         | 0cac10bf2056482cbafde6f696a58f40     |
+-------------------+--------------------------------------+
root@controller:~# neutron security-group-rule-create --direction ingress --protocol tcp --port-range-min 443 --port-range-max 443 --remote-ip-prefix 0.0.0.0/0 lbaas
Created a new security_group_rule:
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| direction         | ingress                              |
| ethertype         | IPv4                                 |
| id                | 82e13910-a40d-4084-929b-eaa0c5b5d0c2 |
| port_range_max    | 443                                  |
| port_range_min    | 443                                  |
| protocol          | tcp                                  |
| remote_group_id   |                                      |
| remote_ip_prefix  | 0.0.0.0/0                            |
| security_group_id | 34bc14b0-fc25-48ca-92c4-2b10c1d940a6 |
| tenant_id         | 0cac10bf2056482cbafde6f696a58f40     |
+-------------------+--------------------------------------+
root@controller:~# neutron security-group-rule-create --direction ingress --protocol icmp lbaas
Created a new security_group_rule:
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| direction         | ingress                              |
| ethertype         | IPv4                                 |
| id                | 0c1c1fd6-17ac-4fa4-951b-7a830a2efddc |
| port_range_max    |                                      |
| port_range_min    |                                      |
| protocol          | icmp                                 |
| remote_group_id   |                                      |
| remote_ip_prefix  |                                      |
| security_group_id | 34bc14b0-fc25-48ca-92c4-2b10c1d940a6 |
| tenant_id         | 0cac10bf2056482cbafde6f696a58f40     |
+-------------------+--------------------------------------+
```

\
使用neutron lbaas-loadbalancer-show命令的vip\_port\_id将安全组应用于负载均衡器的网络端口：\


```
root@controller:~# neutron port-update --security-group lbaas 6b5dfa29-03bd-4af7-b41e-5c2de2360304
Updated port: 6b5dfa29-03bd-4af7-b41e-5c2de2360304
```

\
此负载平衡器处于活动状态，随时可以在10.0.0.100上提供流量。 \
命令：\


```
neutron security-group-create lbaas
neutron security-group-rule-create --direction ingress --protocol tcp --port-range-min 80 --port-range-max 80 --remote-ip-prefix 0.0.0.0/0 lbaas
neutron security-group-rule-create --direction ingress --protocol tcp --port-range-min 443 --port-range-max 443 --remote-ip-prefix 0.0.0.0/0 lbaas
neutron security-group-rule-create --direction ingress --protocol icmp lbaas
neutron port-update --security-group lbaas 6b5dfa29-03bd-4af7-b41e-5c2de2360304
```

**3.2.3.2 添加一个http侦听器**

1.在负载平衡器联机的情况下，您可以为端口80上的明文HTTP流量添加侦听器：\


```
root@controller:~# neutron lbaas-listener-create --name lber-http --loadbalancer lber --protocol HTTP --protocol-port 80
Created a new listener:
+---------------------------+------------------------------------------------+
| Field                     | Value                                          |
+---------------------------+------------------------------------------------+
| admin_state_up            | True                                           |
| connection_limit          | -1                                             |
| default_pool_id           |                                                |
| default_tls_container_ref |                                                |
| description               |                                                |
| id                        | a179a39a-4a19-470e-9a89-c4d57bd8fc4d           |
| loadbalancers             | {"id": "628b6622-154b-4342-8be1-cd645dbb601e"} |
| name                      | lber-http                                      |
| protocol                  | HTTP                                           |
| protocol_port             | 80                                             |
| sni_container_refs        |                                                |
| tenant_id                 | 0cac10bf2056482cbafde6f696a58f40               |
+---------------------------+------------------------------------------------+
root@controller:~# neutron lbaas-listener-list
+--------------------------------------+-----------------+-----------+----------+---------------+----------------+
| id                                   | default_pool_id | name      | protocol | protocol_port | admin_state_up |
+--------------------------------------+-----------------+-----------+----------+---------------+----------------+
| a179a39a-4a19-470e-9a89-c4d57bd8fc4d |                 | lber-http | HTTP     |            80 | True           |
+--------------------------------------+-----------------+-----------+----------+---------------+----------------+
```

\
This load balancer is active and ready to serve traffic on10.0.0.100.\
此时，在网络节点会生成命名空间：qlbaas-628b6622-154b-4342-8be1-cd645dbb601e\


```
root@network:~# ip netns
qlbaas-628b6622-154b-4342-8be1-cd645dbb601e
qrouter-b1461108-8f9a-4746-bf48-6ba717608b34
qdhcp-d1389adb-3fd8-47d2-a8e2-e5978701d33a
qdhcp-1beea70b-6c68-442a-9e87-bdde36bf3092
root@network:~# ip netns exec qlbaas-628b6622-154b-4342-8be1-cd645dbb601e
No command specified
root@network:~# ip netns exec qlbaas-628b6622-154b-4342-8be1-cd645dbb601e ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ns-6b5dfa29-03@if13: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether fa:16:3e:45:3e:30 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.100/24 brd 10.0.0.255 scope global ns-6b5dfa29-03
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe45:3e30/64 scope link 
       valid_lft forever preferred_lft forever
root@network:~# ip netns exec qrouter-b1461108-8f9a-4746-bf48-6ba717608b34 ping 10.0.0.100
PING 10.0.0.100 (10.0.0.100) 56(84) bytes of data.
64 bytes from 10.0.0.100: icmp_seq=1 ttl=64 time=0.133 ms
64 bytes from 10.0.0.100: icmp_seq=2 ttl=64 time=0.059 ms
^C
--- 10.0.0.100 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.059/0.096/0.133/0.037 ms
```

\
2.您可以开始构建池，并向池中添加成员以在端口80上提供HTTP内容。对于此示例，Web服务器为10.0.0.102和10.0.0.103：\


```
root@controller:~# nova list
+--------------------------------------+------+--------+------------+-------------+-----------------------------------+
| ID                                   | Name | Status | Task State | Power State | Networks                          |
+--------------------------------------+------+--------+------------+-------------+-----------------------------------+
| fc1afaa9-a1cf-40f5-a563-eb95980f8e22 | aaa  | ACTIVE | -          | Running     | private=10.0.0.102, 192.168.4.141 |
| 9cd6042a-521d-4578-b1c6-026777ed72eb | bbb  | ACTIVE | -          | Running     | private=10.0.0.103, 192.168.4.142 |
+--------------------------------------+------+--------+------------+-------------+-----------------------------------+
```

\
在服务器aaa和bbb上启动http服务：\
方法一：执行如下命令添加一个80端口的监听进程，模拟httpd监听\


```
root@aaa:~# while true; do echo -e "HTTP/1.0 200 OK\r\n\r\nWelcome to aaa" | nc -l -p 80 ; done& 
```

方法二：\


```
root@aaa:~# echo "Welcome to aaa" >index.html
root@aaa:~# setsid python -m SimpleHTTPServer 80
root@aaa:~# ps -ef | grep 80
root 2654 1 0 08:15 pts/0 00:00:00 python -m SimpleHTTPServer 80
```

方法三：安装并启动nginx服务：\


```
root@aaa:~# apt-get install nginx
root@aaa:~# ps -ef | grep nginx
root      3659     1  0 03:51 ?        00:00:00 nginx: master process /usr/sbin/nginx
www-data  3660  3659  0 03:51 ?        00:00:00 nginx: worker process
www-data  3661  3659  0 03:51 ?        00:00:00 nginx: worker process
www-data  3662  3659  0 03:51 ?        00:00:00 nginx: worker process
www-data  3663  3659  0 03:51 ?        00:00:00 nginx: worker process
root@aaa:~# ss -ant | grep 80
LISTEN     0      128                       *:80                       *:* 
LISTEN     0      128                      :::80                      :::* 
```

\
创建pool：\
root@controller:\~# neutron lbaas-pool-create --name lber-pool-http --lb-algorithm ROUND\_ROBIN --listener lber-http --protocol HTTP\


```
Created a new pool:
+---------------------+------------------------------------------------+
| Field               | Value                                          |
+---------------------+------------------------------------------------+
| admin_state_up      | True                                           |
| description         |                                                |
| healthmonitor_id    |                                                |
| id                  | b0948653-26e1-454d-923f-019107c0dc4d           |
| lb_algorithm        | ROUND_ROBIN                                    |
| listeners           | {"id": "a179a39a-4a19-470e-9a89-c4d57bd8fc4d"} |
| members             |                                                |
| name                | lber-pool-http                                 |
| protocol            | HTTP                                           |
| session_persistence |                                                |
| tenant_id           | 0cac10bf2056482cbafde6f696a58f40               |
+---------------------+------------------------------------------------+
root@controller:~# neutron lbaas-member-create --subnet private --address 10.0.0.102 --protocol-port 80 lber-pool-http
Created a new member:
+----------------+--------------------------------------+
| Field          | Value                                |
+----------------+--------------------------------------+
| address        | 10.0.0.102                           |
| admin_state_up | True                                 |
| id             | ef90b8ef-6dfd-40af-885e-0eb0b4bbcf1b |
| protocol_port  | 80                                   |
| subnet_id      | 6b9c6742-f965-4b62-899b-6e60da743e66 |
| tenant_id      | 0cac10bf2056482cbafde6f696a58f40     |
| weight         | 1                                    |
+----------------+--------------------------------------+
root@controller:~# neutron lbaas-member-create --subnet private --address 10.0.0.103 --protocol-port 80 lber-pool-http
Created a new member:
+----------------+--------------------------------------+
| Field          | Value                                |
+----------------+--------------------------------------+
| address        | 10.0.0.103                           |
| admin_state_up | True                                 |
| id             | 25152e25-e2d9-4fc1-94f5-4f6ac95e6e64 |
| protocol_port  | 80                                   |
| subnet_id      | 6b9c6742-f965-4b62-899b-6e60da743e66 |
| tenant_id      | 0cac10bf2056482cbafde6f696a58f40     |
| weight         | 1                                    |
+----------------+--------------------------------------+
root@controller:~# neutron lbaas-member-list lber-pool-http
+--------------------------------------+------------+---------------+--------+--------------------------------------+----------------+
| id                                   | address    | protocol_port | weight | subnet_id                            | admin_state_up |
+--------------------------------------+------------+---------------+--------+--------------------------------------+----------------+
| ef90b8ef-6dfd-40af-885e-0eb0b4bbcf1b | 10.0.0.102 |            80 |      1 | 6b9c6742-f965-4b62-899b-6e60da743e66 | True           |
| 25152e25-e2d9-4fc1-94f5-4f6ac95e6e64 | 10.0.0.103 |            80 |      1 | 6b9c6742-f965-4b62-899b-6e60da743e66 | True           |
+--------------------------------------+------------+---------------+--------+--------------------------------------+----------------+
```

\
3.您可以使用curl验证通过负载平衡器到您的Web服务器的连接：\


```
root@network:~# ip netns exec qrouter-b1461108-8f9a-4746-bf48-6ba717608b34 curl 10.0.0.100
<html><body><h1>503 Service Unavailable</h1>
No server is available to handle this request.
</body></html>
```

此问题是没有向pool中添加成员。用方法一启动http服务成功。重新生成新的虚拟机成员后，curl结果：

```
root@network:~# ip net exec qrouter-b1461108-8f9a-4746-bf48-6ba717608b34 curl 10.0.0.100
Welcome to vm1!
root@network:~# ip net exec qrouter-b1461108-8f9a-4746-bf48-6ba717608b34 curl 10.0.0.100
Welcome to vm2!
root@network:~# ip net exec qrouter-b1461108-8f9a-4746-bf48-6ba717608b34 curl 10.0.0.100
Welcome to vm1!
root@network:~# ip net exec qrouter-b1461108-8f9a-4746-bf48-6ba717608b34 curl 10.0.0.100
Welcome to vm2!
```

\
在此示例中，负载平衡器使用轮转算法和后端的Web服务器之间的流量交替。 \
4.您可以添加运行状况监视器，以便从池中除去无响应的服务器：\


```
root@controller:~# neutron lbaas-healthmonitor-create --delay 5 --max-retries 2 --timeout 10 --type HTTP --pool lber-pool-http
Created a new healthmonitor:
+----------------+------------------------------------------------+
| Field          | Value                                          |
+----------------+------------------------------------------------+
| admin_state_up | True                                           |
| delay          | 5                                              |
| expected_codes | 200                                            |
| http_method    | GET                                            |
| id             | acf9bb3c-5d85-4a01-a601-252b1a53de3e           |
| max_retries    | 2                                              |
| pools          | {"id": "b0948653-26e1-454d-923f-019107c0dc4d"} |
| tenant_id      | 0cac10bf2056482cbafde6f696a58f40               |
| timeout        | 10                                             |
| type           | HTTP                                           |
| url_path       | /                                              |
+----------------+------------------------------------------------+
root@controller:~# neutron lbaas-healthmonitor-list
+--------------------------------------+------+----------------+
| id                                   | type | admin_state_up |
+--------------------------------------+------+----------------+
| 3b3eff3c-3453-4946-8fe4-c9a1b8b45af9 | HTTP | True           |
| acf9bb3c-5d85-4a01-a601-252b1a53de3e | HTTP | True           |
+--------------------------------------+------+----------------+
root@controller:~# neutron lbaas-healthmonitor-show acf9bb3c-5d85-4a01-a601-252b1a53de3e
+----------------+------------------------------------------------+
| Field          | Value                                          |
+----------------+------------------------------------------------+
| admin_state_up | True                                           |
| delay          | 5                                              |
| expected_codes | 200                                            |
| http_method    | GET                                            |
| id             | acf9bb3c-5d85-4a01-a601-252b1a53de3e           |
| max_retries    | 2                                              |
| pools          | {"id": "b0948653-26e1-454d-923f-019107c0dc4d"} |
| tenant_id      | 0cac10bf2056482cbafde6f696a58f40               |
| timeout        | 10                                             |
| type           | HTTP                                           |
| url_path       | /                                              |
+----------------+------------------------------------------------+
```

在此示例中，如果运行状况监视器以两个5秒的间隔时间未通过运行状况检查，则会从池中删除服务器。 当服务器恢复并再次开始响应运行状况检查时，它会再次添加到池中。\


**3.2.3.3 添加一个https侦听器**

您可以在端口443上为HTTPS通信添加另一个侦听器。 LBaaS v2在负载均衡器上提供SSL / TLS终止，但此示例采用更简单的方法，并允许加密连接在每个成员服务器上终止。 \


```
neutron lbaas-listener-create --name lber-https --loadbalancer lber --protocol HTTPS --protocol-port 443
neutron lbaas-pool-create --name lber-pool-https --lb-algorithm LEAST_CONNECTIONS --listener lber-https --protocol HTTPS
neutron lbaas-member-create --subnet private --address 10.0.0.102 --protocol-port 443 lber-pool-https
neutron lbaas-member-create --subnet private --address 10.0.0.103 --protocol-port 443 lber-pool-https
```

你也可以为https池添加一个健康监视器\
neutron lbaas-healthmonitor-create --delay 5 --max-retries 2 --timeout 10 --type HTTPS --pool lber-pool-https\
负载均衡器现在控制着80和443端口的流量。\


**3.2.3.4 添加一个https侦听器**

1 创建listener、pool、member，添加一个健康监视器\


```
root@controller:~# neutron lbaas-listener-create --name lber-ssh --loadbalancer lber --protocol TCP --protocol-port 22
Created a new listener:
+---------------------------+------------------------------------------------+
| Field                     | Value                                          |
+---------------------------+------------------------------------------------+
| admin_state_up            | True                                           |
| connection_limit          | -1                                             |
| default_pool_id           |                                                |
| default_tls_container_ref |                                                |
| description               |                                                |
| id                        | aaa05386-b8d6-4c51-84d1-21d525c8219a           |
| loadbalancers             | {"id": "628b6622-154b-4342-8be1-cd645dbb601e"} |
| name                      | lber-ssh                                       |
| protocol                  | TCP                                            |
| protocol_port             | 22                                             |
| sni_container_refs        |                                                |
| tenant_id                 | 0cac10bf2056482cbafde6f696a58f40               |
+---------------------------+------------------------------------------------+
root@controller:~# neutron lbaas-pool-create --name lber-pool-ssh --lb-algorithm ROUND_ROBIN --listener lber-ssh --protocol TCP  
Created a new pool:
+---------------------+------------------------------------------------+
| Field               | Value                                          |
+---------------------+------------------------------------------------+
| admin_state_up      | True                                           |
| description         |                                                |
| healthmonitor_id    |                                                |
| id                  | 7efcaf01-0b6c-4bf2-aa07-851f26379083           |
| lb_algorithm        | ROUND_ROBIN                                    |
| listeners           | {"id": "aaa05386-b8d6-4c51-84d1-21d525c8219a"} |
| members             |                                                |
| name                | lber-pool-ssh                                  |
| protocol            | TCP                                            |
| session_persistence |                                                |
| tenant_id           | 0cac10bf2056482cbafde6f696a58f40               |
+---------------------+------------------------------------------------+
root@controller:~# neutron lbaas-member-create --subnet private --address 10.0.0.102 --protocol-port 22 lber-pool-ssh
Created a new member:
+----------------+--------------------------------------+
| Field          | Value                                |
+----------------+--------------------------------------+
| address        | 10.0.0.102                           |
| admin_state_up | True                                 |
| id             | 8f3fe899-6b1f-479a-8f24-78a2096991d8 |
| protocol_port  | 22                                   |
| subnet_id      | 6b9c6742-f965-4b62-899b-6e60da743e66 |
| tenant_id      | 0cac10bf2056482cbafde6f696a58f40     |
| weight         | 1                                    |
+----------------+--------------------------------------+
root@controller:~# neutron lbaas-member-create --subnet private --address 10.0.0.103 --protocol-port 22 lber-pool-ssh
Created a new member:
+----------------+--------------------------------------+
| Field          | Value                                |
+----------------+--------------------------------------+
| address        | 10.0.0.103                           |
| admin_state_up | True                                 |
| id             | 3b842a8b-fc5d-4ebe-8e89-6cdf02ac7183 |
| protocol_port  | 22                                   |
| subnet_id      | 6b9c6742-f965-4b62-899b-6e60da743e66 |
| tenant_id      | 0cac10bf2056482cbafde6f696a58f40     |
| weight         | 1                                    |
+----------------+--------------------------------------+
root@controller:~# neutron lbaas-healthmonitor-create --delay 5 --max-retries 2 --timeout 10 --type TCP --pool lber-pool-ssh
Created a new healthmonitor:
+----------------+------------------------------------------------+
| Field          | Value                                          |
+----------------+------------------------------------------------+
| admin_state_up | True                                           |
| delay          | 5                                              |
| expected_codes | 200                                            |
| http_method    | GET                                            |
| id             | 62f0b5cd-fb05-4a7f-aeea-52d34bf5e2c7           |
| max_retries    | 2                                              |
| pools          | {"id": "7efcaf01-0b6c-4bf2-aa07-851f26379083"} |
| tenant_id      | 0cac10bf2056482cbafde6f696a58f40               |
| timeout        | 10                                             |
| type           | TCP                                            |
| url_path       | /                                              |
+----------------+------------------------------------------------+
root@controller:~# neutron lbaas-healthmonitor-list
+--------------------------------------+-------+----------------+
| id                                   | type  | admin_state_up |
+--------------------------------------+-------+----------------+
| 3b3eff3c-3453-4946-8fe4-c9a1b8b45af9 | HTTP  | True           |
| 62f0b5cd-fb05-4a7f-aeea-52d34bf5e2c7 | TCP   | True           |
| 69bc9a6a-fbdf-4a9d-b462-53f87a7fbe2b | HTTPS | True           |
| acf9bb3c-5d85-4a01-a601-252b1a53de3e | HTTP  | True           |
+--------------------------------------+-------+----------------+
```

2 通过LoadBalancer访问ssh server\
在网络节点访问，第一次连上aaa服务器\


```
root@network:~# ip net exec qrouter-b1461108-8f9a-4746-bf48-6ba717608b34 ssh 10.0.0.100
...
Last login: Tue Jul 11 03:46:53 2017 from 192.168.4.131
root@aaa:~#
```

第二次连上bbb服务器\


```
root@network:~# ssh-keygen -f "/root/.ssh/known_hosts" -R 10.0.0.100
# Host 10.0.0.100 found: line 5 type ECDSA
/root/.ssh/known_hosts updated.
Original contents retained as /root/.ssh/known_hosts.old
root@network:~# ip net exec qrouter-b1461108-8f9a-4746-bf48-6ba717608b34 ssh 10.0.0.100
...
Last login: Tue Jul 11 03:46:53 2017 from 192.168.4.131
root@bbb:~#
```

**3.2.3.5 关联浮动IP地址**

部署在公用或提供商网络上的外部客户端可访问的负载平衡器不需要分配浮动IP地址。 外部客户端可以直接访问这些负载平衡器的虚拟IP地址（VIP）。 \
但是，部署到专用或隔离网络上的负载平衡器需要分配浮动IP地址，如果它们必须可由外部客户端访问。 要完成此步骤，您必须在私有和公共网络之间有一个路由器和一个可用的浮动IP地址。 \
您可以使用本节开头的neutron lbaas-loadbalancer-show命令来查找vip\_port\_id。 vip\_port\_id是分配给负载平衡器的网络端口的ID。 您可以使用neutron floatingip-associate将自由浮动IP地址与负载均衡器关联：\


```
$ neutron floatingip-associate FLOATINGIP_ID LOAD_BALANCER_PORT_ID
root@controller:~# neutron lbaas-loadbalancer-show lber
+---------------------+------------------------------------------------+
| Field               | Value                                          |
+---------------------+------------------------------------------------+
| admin_state_up      | True                                           |
| description         |                                                |
| id                  | 628b6622-154b-4342-8be1-cd645dbb601e           |
| listeners           | {"id": "9223af16-23e0-48ea-9d5e-6069581cf97b"} |
|                     | {"id": "aaa05386-b8d6-4c51-84d1-21d525c8219a"} |
|                     | {"id": "a179a39a-4a19-470e-9a89-c4d57bd8fc4d"} |
| name                | lber                                           |
| operating_status    | ONLINE                                         |
| provider            | haproxy                                        |
| provisioning_status | ACTIVE                                         |
| tenant_id           | 0cac10bf2056482cbafde6f696a58f40               |
| vip_address         | 10.0.0.100                                     |
| vip_port_id         | 6b5dfa29-03bd-4af7-b41e-5c2de2360304           |
| vip_subnet_id       | 6b9c6742-f965-4b62-899b-6e60da743e66           |
+---------------------+------------------------------------------------+
```

\
关联floating ip后，直接ssh 192.168.4.144，可连接上aaa或bbb服务器\


```
root@network:~# ssh 192.168.4.144
The authenticity of host '192.168.4.144 (192.168.4.144)' can't be established.
ECDSA key fingerprint is 7e:8e:0c:b7:03:ab:91:21:62:75:aa:43:89:6c:ea:a4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.4.144' (ECDSA) to the list of known hosts.
root@192.168.4.144's password: 
...
Last login: Tue Jul 11 08:49:57 2017 from 10.0.0.100
root@bbb:~# logout
Connection to 192.168.4.144 closed.
root@network:~# ssh-keygen -f "/root/.ssh/known_hosts" -R 192.168.4.144
# Host 192.168.4.144 found: line 5 type ECDSA
/root/.ssh/known_hosts updated.
Original contents retained as /root/.ssh/known_hosts.old
root@network:~# 
root@network:~# ssh 192.168.4.144
The authenticity of host '192.168.4.144 (192.168.4.144)' can't be established.
ECDSA key fingerprint is b8:37:81:d8:c2:67:26:4d:49:96:a2:86:ca:40:95:cd.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.4.144' (ECDSA) to the list of known hosts.
root@192.168.4.144's password: 
...
Last login: Tue Jul 11 08:49:38 2017 from 10.0.0.100
root@aaa:~# logout
Connection to 192.168.4.144 closed.
```

**3.2.3.6 为LBaaS v2 设置配额**

配额可用于限制负载平衡器和负载平衡器池的数量。 默认情况下，两个配额都设置为10。 \
您可以使用neutron quota-update命令调整配额：\
neutron quota-update --tenant-id TENANT\_UUID --loadbalancer 25\
neutron quota-update --tenant-id TENANT\_UUID --pool 50\
设置为-1会禁用租户的配额。\


**3.2.3.7 检索负载平衡器统计信息**

LBaaS v2代理每6秒为每个负载平衡器收集四种类型的统计信息。 用户可以使用neutron lbaas-loadbalancer-stats命令查询这些统计信息：\


```
$ neutron lbaas-loadbalancer-stats test-lb
+--------------------+----------+
| Field              | Value    |
+--------------------+----------+
| active_connections | 0        |
| bytes_in           | 40264557 |
| bytes_out          | 71701666 |
| total_connections  | 384601   |
+--------------------+----------+
```

\
active\_connections计数是代理轮询负载平衡器时处于活动状态的连接总数。 自上次启动负载平衡器以来，其他三个统计信息是累积的。 例如，如果负载平衡器由于系统错误或配置更改而重新启动，则这些统计信息将被重置。\
\


## 4 实现机制

对于每一个 loadbalancer，Neutron 都会启动一个 haproxy 进程提供 load balancering 功能。 \
通过 ps 命令查找 haproxy 进程：\


```
root@network:~# ps -ef | grep haproxy
nobody   11368     1  0 15:47 ?        00:00:00 haproxy -f /var/lib/neutron/lbaas/v2/628b6622-154b-4342-8be1-cd645dbb601e/haproxy.conf -p /var/lib/neutron/lbaas/v2/628b6622-154b-4342-8be1-cd645dbb601e/haproxy.pid -sf 11361
```

haproxy 配置文件保存在 /opt/stack/data/neutron/lbaas/< pool ID>/conf 中。 \
查看 “web servers” 的配置内容：\


```
root@network:~# cat /var/lib/neutron/lbaas/v2/628b6622-154b-4342-8be1-cd645dbb601e/haproxy.conf
# Configuration for lber
global
    daemon
    user nobody
    group haproxy
    log /dev/log local0
    log /dev/log local1 notice
    stats socket /var/lib/neutron/lbaas/v2/628b6622-154b-4342-8be1-cd645dbb601e/haproxy_stats.sock mode 0666 level user

defaults
    log global
    retries 3
    option redispatch
    timeout connect 5000
    timeout client 50000
    timeout server 50000

frontend 1d99596e-2689-466f-86cd-a3ff3f65884a
    option tcplog
    option forwardfor
    bind 10.0.0.100:80
    mode http
    default_backend 9ab6caeb-c895-4d24-9ba6-1f91717c753f

backend 9ab6caeb-c895-4d24-9ba6-1f91717c753f
    mode http
    balance roundrobin
    option forwardfor
    server 559fcd99-ef9e-4be4-ad4d-71bc57e212d7 10.0.0.102:80 weight 1
    server c17be654-04f0-4796-9f59-2c3d90602da6 10.0.0.103:80 weight 1

frontend aaa05386-b8d6-4c51-84d1-21d525c8219a
    option tcplog
    bind 10.0.0.100:22
    mode tcp
    default_backend 7efcaf01-0b6c-4bf2-aa07-851f26379083

backend 7efcaf01-0b6c-4bf2-aa07-851f26379083
    mode tcp
    balance roundrobin
    timeout check 10
    server 3b842a8b-fc5d-4ebe-8e89-6cdf02ac7183 10.0.0.103:22 weight 1 check inter 5s fall 2
    server 8f3fe899-6b1f-479a-8f24-78a2096991d8 10.0.0.102:22 weight 1 check inter 5s fall 2

frontend 9223af16-23e0-48ea-9d5e-6069581cf97b
    option tcplog
    bind 10.0.0.100:443
    mode tcp
    default_backend 7bfd6d4f-6676-4eea-b319-99837dc335da

backend 7bfd6d4f-6676-4eea-b319-99837dc335da
    mode tcp
    balance leastconn
    timeout check 10
    option httpchk GET /
    http-check expect rstatus 200
    option ssl-hello-chk
    server 962cea25-ecb7-41b8-9eed-ca27c20621d4 10.0.0.103:443 weight 1 check inter 5s fall 2
    server 7fcab79c-dc99-41e4-9218-437f5c59841b 10.0.0.102:443 weight 1 check inter 5s fall 2
```

\
可以看到： \
1\. frontend 使用的 HTTP 地址为 VIP:80 \
2\. backend 使用的 HTTP 地址为 10.0.0.102:80 和 10.0.0.103:80 \
3\. balance 方法为 roundrobin

## References

原文 [OpenStack LBaaS之LBaaS](https://blog.csdn.net/zhaihaifei/article/details/74732547)

1  [http://blog.csdn.net/fyggzb/article/details/53924976](http://blog.csdn.net/fyggzb/article/details/53924976)

2  [https://docs.openstack.org/ocata/networking-guide/config-lbaas.html](https://docs.openstack.org/ocata/networking-guide/config-lbaas.html)

3  [https://wiki.openstack.org/wiki/Neutron/LBaaS](https://wiki.openstack.org/wiki/Neutron/LBaaS)

4  [http://blog.csdn.net/CloudMan6/article/details/53613152](http://blog.csdn.net/CloudMan6/article/details/53613152)

5  [http://www.cnblogs.com/sammyliu/p/4656176.html](http://www.cnblogs.com/sammyliu/p/4656176.html)
