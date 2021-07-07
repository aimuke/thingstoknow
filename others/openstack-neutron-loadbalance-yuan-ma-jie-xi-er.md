# OpenStack Neutron LoadBalance源码解析（二）

作者：林凯

团队：华为杭州OpenStack团队

在[Neutron LoadBalance源码解析（一）](http://blog.csdn.net/canxinghen/article/details/42143809)中，我们已经了解租户在创建pool、member、healthmonitor和vip的时候，代码会调用HaproxyNSDriver中的create\_xxx函数，那么当租户使用pool创建vip时，即代码调用HaproxyNSDriver中的create\_vip函数时，Neutron LoadBalancer 的基本部署已经完成，那么create\_vip这个方法具体做了哪些事情呢？我们一起继续往下看：

## HaproxyNSDriver create\_vip

HaproxyNSDriver中的create\_vip：

```python
def create_vip(self, vip):
        self._refresh_device(vip['pool_id'])
```

```python
# 获取配置，然后根据配置部署实例
def _refresh_device(self, pool_id):
		logical_config = self.plugin_rpc.get_logical_device(pool_id)
        self.deploy_instance(logical_config)
```

```python
def deploy_instance(self, logical_config):
        # do actual deploy only if vip and pool are configured and active
        # 如果vip和pool被正确配置并且为active
        if (not logical_config or
                'vip' not in logical_config or
                (logical_config['vip']['status'] not in
                 constants.ACTIVE_PENDING_STATUSES) or
                not logical_config['vip']['admin_state_up'] or
                (logical_config['pool']['status'] not in
                 constants.ACTIVE_PENDING_STATUSES) or
                not logical_config['pool']['admin_state_up']):
            return

        if self.exists(logical_config['pool']['id']):
            self.update(logical_config)
        else:
            self.create(logical_config)
```

我们可以看到只有当vip和pool均被正确配置，且状态为active的时候，才会执行真正的操作。这个时候pool还没有被真正建立，所以执行的是create操作。

```python
def create(self, logical_config):
        pool_id = logical_config['pool']['id']
        namespace = get_ns_name(pool_id)

        self._plug(namespace, logical_config['vip']['port'])
        self._spawn(logical_config)
```



这边有两个重要的操作：plug和spawn，分别来看下：

## plug

```python
def _plug(self, namespace, port, reuse_existing=True):
        # 更新DB vip创建时创建的port的信息
        self.plugin_rpc.plug_vip_port(port['id'])
        interface_name = self.vif_driver.get_device_name(Wrap(port))

        #判断在命名空间是否存在设备，存在则跳过
        #不存在这个设备，利用vif_driver进行plug
        if ip_lib.device_exists(interface_name, self.root_helper, namespace):
            if not reuse_existing:
                raise exceptions.PreexistingDeviceFailure(
                    dev_name=interface_name
                )
        else:
            self.vif_driver.plug(
                port['network_id'],
                port['id'],
                interface_name,
                port['mac_address'],
                namespace=namespace
            )

        cidrs = [
            '%s/%s' % (ip['ip_address'],
                       netaddr.IPNetwork(ip['subnet']['cidr']).prefixlen)
            for ip in port['fixed_ips']
        ]
        # 为网卡设置L3初始化
        self.vif_driver.init_l3(interface_name, cidrs, namespace=namespace)

        gw_ip = port['fixed_ips'][0]['subnet'].get('gateway_ip')

        # 分别出有网关IP和无网关IP的情况
        if not gw_ip:
            host_routes = port['fixed_ips'][0]['subnet'].get('host_routes', [])
            for host_route in host_routes:
                if host_route['destination'] == "0.0.0.0/0":
                    gw_ip = host_route['nexthop']
                    break

        if gw_ip:
            # 设置默认网关IP
            cmd = ['route', 'add', 'default', 'gw', gw_ip]
            ip_wrapper = ip_lib.IPWrapper(self.root_helper,
                                          namespace=namespace)
            ip_wrapper.netns.execute(cmd, check_exit_code=False)
            # When delete and re-add the same vip, we need to
            # send gratuitous ARP to flush the ARP cache in the Router.
            # 当删除或重添加同样的vip时，我们需要发送gratuitous ARP
            gratuitous_arp = self.conf.haproxy.send_gratuitous_arp
            if gratuitous_arp > 0:
                for ip in port['fixed_ips']:
                    cmd_arping = ['arping', '-U',
                                  '-I', interface_name,
                                  '-c', gratuitous_arp,
                                  ip['ip_address']]
                    ip_wrapper.netns.execute(cmd_arping, check_exit_code=False)
```

从上面的代码中可知，当不存在网络设备时，要利用vif\_driver进行plug，这个plug操作又完成了什么事情呢？因为默认的driver是OVSInterfacedriver，所以在/agent/linux/interface.py中找到OVSInterfaceDriver的plug方法：

```python
def plug(self, network_id, port_id, device_name, mac_address,
             bridge=None, namespace=None, prefix=None):
        """Plug in the interface."""
        if not bridge:
            bridge = self.conf.ovs_integration_bridge

        if not ip_lib.device_exists(device_name,
                                    self.root_helper,
                                    namespace=namespace):

            self.check_bridge_exists(bridge)

            ip = ip_lib.IPWrapper(self.root_helper)
            # 获取设备信息，采用tap设备或者veth设备
            # TAP 设备是一种让用户态程序向内核协议栈注入数据的设备
            # VETH 作用是反转通讯数据的方向，需要发送的数据会被转换成需要收到的数据
            # 重新送入内核网络层进行处理，从而间接的完成数据的注入
            tap_name = self._get_tap_name(device_name, prefix)

            if self.conf.ovs_use_veth:
                # Create ns_dev in a namespace if one is configured.
                root_dev, ns_dev = ip.add_veth(tap_name,
                                               device_name,
                                               namespace2=namespace)
            else:
                ns_dev = ip.device(device_name)

            # 如果br-int没有问题，创建网卡，并且attach到br-int上
            # 如果用的是veth设备，则不添加网卡为internal类型接口。
            internal = not self.conf.ovs_use_veth
            self._ovs_add_port(bridge, tap_name, port_id, mac_address,
                               internal=internal)
            # 为网卡设置mac_address
            # ip link set tap452bdfab-31 address fa:16:3e:d7:08:67
            ns_dev.link.set_address(mac_address)

            # 设置mtu
            if self.conf.network_device_mtu:
                ns_dev.link.set_mtu(self.conf.network_device_mtu)
                if self.conf.ovs_use_veth:
                    root_dev.link.set_mtu(self.conf.network_device_mtu)

            # Add an interface created by ovs to the namespace.
            # 将新建的网卡放在这个namespace里面
            if not self.conf.ovs_use_veth and namespace:
                namespace_obj = ip.ensure_namespace(namespace)
                namespace_obj.add_device_to_namespace(ns_dev)

            # 将网卡设置为up
            ns_dev.link.set_up()
            if self.conf.ovs_use_veth:
                root_dev.link.set_up()
        else:
            LOG.info(_("Device %s already exists"), device_name)
```

这里的plug操作可以这样理解： LB的这些设备需要连接到交换机（指ovs虚拟交换机：br-int）上才能与网络相通，并正常工作。而要与网络相通，我们需要为设备添加一张网卡，并且把网卡attach到交换机上，然后配置正确，这些设备才能正常工作。

在代码中还可以看到将网卡放在namespace这个东西中，namespace又是用来做什么的？感兴趣的可以看下：[Linux Namespaces机制](http://blog.csdn.net/preterhuman_peak/article/details/40857117) ，简而言之：namespace提供了一个容器，它为多个进程提供了一个完全独立的网络协议栈的视图。包括网络设备接口，IPv4和IPv6协议栈，IP路由表，防火墙规则，sockets等。所以网卡放入到namespace中，这个网络设备接口就被进程所感知并可被使用。

进行plug操作之后，还需要为网卡进行l3的初始化，并设置默认网关IP，这里我们很多人就有疑问：为什么要进行这些设置呢？这边就涉及到Neutron中实现L4/L7层服务的框架Service Insertion。

Neutron又实现了一层叫做“服务”的插件结构，即现在有两层插件结构，在NeutronPlugin上又可以启动多个服务，neutronPlugin的服务插件继续和数据库打交道同时也能共享原有的NeutronPlugin的信息，如port信息；同时，像FWaaS服务要求ServiceAgent可以运行在l3-agent所在的网络节点上，而LBaaS的haproxy并不需要也安装在l3-agent，但l3-agent也应该在一个专门的命名空间里创建一个port和haproxy所在的host是联通的。

所以LBaaS服务不需要运行在网络节点上，但网络节点上专门为它准备的port和实际运行haproxy的节点应该通，这个port和网关也应该通，即所谓的Floating/In-Path模式。所以才有了以上的操作与配置。

![](../.gitbook/assets/image%20%2845%29.png)

到这里\_plug操作已经完成了，然后还有一个坑没填，就是\_spawn方法中执行了哪些操作呢？

## spawn

```python
def _spawn(self, logical_config, extra_cmd_args=()):
        pool_id = logical_config['pool']['id']
        namespace = get_ns_name(pool_id)
        conf_path = self._get_state_file_path(pool_id, 'conf')
        pid_path = self._get_state_file_path(pool_id, 'pid')
        sock_path = self._get_state_file_path(pool_id, 'sock')
        user_group = self.conf.haproxy.user_group

        hacfg.save_config(conf_path, logical_config, sock_path, user_group)
        cmd = ['haproxy', '-f', conf_path, '-p', pid_path]
        cmd.extend(extra_cmd_args)

        ns = ip_lib.IPWrapper(self.root_helper, namespace)
        ns.netns.execute(cmd)

        # remember the pool<>port mapping
        self.pool_to_port_id[pool_id] = logical_config['vip']['port']['id']
```

这里的操作是根据之前的配置文件和设备信息，生成新的配置文件，写入到haproxy中使其生效。这样子整个LBaaS就可以利用haproxy进行负载均衡。

至此，LBaaS V1.0的模块解析和源代码分析就告一段落。**时间：** 01-08



## References

* 原文 [OpenStack Neutron LoadBalance源码解析（二）](https://www.biecuoliao.com/pa/0e99Eg0.html)

