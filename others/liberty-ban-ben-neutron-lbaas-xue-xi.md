# Liberty 版本 Neutron LBaas 学习

## 架构

模块示意图

![](<../.gitbook/assets/image (57).png>)

\
这里参考的是 lbaasv2 的 driver，neutron\_lbaas.conf 中的 service\_provider 即为 lbaasv2 的 driver：

```python
service_provider=LOADBALANCERV2:Haproxy:neutron_lbaas.drivers.haproxy.plugin_driver.HaproxyOnHostPluginDriver:default
```

neutron.conf 中的 service\_plugins 表示了 lbaasv2 的 plugin：

```python
lbaasv2 = neutron_lbaas.services.loadbalancer.plugin:LoadBalancerPluginv2
```

### plugin

{% code title="neutron_lbaas.services.loadbalancer.plugin.py" %}
```python
class LoadBalancerPluginv2(loadbalancerv2.LoadBalancerPluginBaseV2):
    def __init__(self):
        """Initialization for the loadbalancer service plugin."""
        self.db = ldbv2.LoadBalancerPluginDbv2()
```
{% endcode %}

### driver

{% code title="neutron_lbaas.drivers.haproxy.plugin_driver.py" %}
```python
class HaproxyOnHostPluginDriver(agent_driver_base.AgentDriverBase):
    device_driver = namespace_driver.DRIVER_NAME
```
{% endcode %}

### agent rpc（plugin 向 agent 发送）

{% code title="neutron_lbaas.drivers.common.agent_driver_base.py" %}
```python
class LoadBalancerAgentApi
    def __init__
```
{% endcode %}

### &#x20;plugin rpc（agent 向 plugin 发送）

{% code title="neutron_lbaas.agent.agent_api.py" %}
```python
class LbaasAgentApi
    def __init__
```
{% endcode %}

### agent 侧的回调

{% code title="neutron_lbaas.agent.agent_manager.py" %}
```python
class LbaasAgentManager
```
{% endcode %}

### plugin 侧的回调

{% code title="neutron_lbaas.drivers.common.agent_driver_base.py" %}
```python
class AgentDriverBase
def _set_callbacks_on_plugin
```
{% endcode %}

{% code title="neutron_lbaas.drivers.common.agent_callbacks.py" %}
```python
class LoadBalancerCallbacks
```
{% endcode %}

### agent 入口函数

{% code title="neutron_lbaas.agent.agent.py" %}
```python
def main
```
{% endcode %}

### db 处理

{% code title="neutron_lbaas.db.loadbalancer.loadbalancer_dbv2.py" %}
```python
class LoadBalancerPluginDbv2
```
{% endcode %}

## 流程示例，创建一个 Pool

## plugin 部分

首先进入一个同步流程，即创建 pool 的数据库信息。

### **plugin 入口**

{% code title="neutron_lbaas.services.loadbalancer.plugin.py" %}
```python
class LoadBalancerPluginv2(loadbalancerv2.LoadBalancerPluginBaseV2):

    def create_pool(self, context, pool):
        try:
            # 创建pool的数据库信息，并绑定listener
            db_pool = self.db.create_pool_and_add_to_listener(context, pool,
                                                              listener_id)
        except Exception as exc:
            self.db.update_loadbalancer_provisioning_status(
                context, db_listener.loadbalancer.id)
            raise exc
        # 根据创建lb时的provider来选择driver
        driver = self._get_driver_for_loadbalancer(
            context, db_pool.listener.loadbalancer_id)
        # 将调用driver的pool.create方法，进入driver的父类
        # AgentDriverBase
        self._call_driver_operation(context, driver.pool.create, db_pool)
        # 同步返回了pool的数据库信息
        return self.db.get_pool(context, db_pool.id).to_api_dict()
```
{% endcode %}

### **plugin 的 api 及 rpc 处理**

{% code title="neutron_lbaas.drivers.common.agent_driver_base.py" %}
```python
class AgentDriverBase(driver_base.LoadBalancerBaseDriver):

    # name of device driver that should be used by the agent;
    # vendor specific plugin drivers must override it;
    device_driver = None

    def __init__(self, plugin):
        super(AgentDriverBase, self).__init__(plugin)
        # pool的管理类
        self.pool = PoolManager(self)
        # agent的rpc处理类
        self.agent_rpc = LoadBalancerAgentApi(lb_const.LOADBALANCER_AGENTV2)
        # rpc的回调
        self._set_callbacks_on_plugin()
        # Setting this on the db because the plugin no longer inherts from
        # database classes, the db does.
        self.plugin.db.agent_notifiers.update(
            {lb_const.AGENT_TYPE_LOADBALANCERV2: self.agent_rpc})

        # agent的调度driver
        lb_sched_driver = provconf.get_provider_driver_class(
            cfg.CONF.loadbalancer_scheduler_driver, LB_SCHEDULERS)
        self.loadbalancer_scheduler = importutils.import_object(
            lb_sched_driver)

class PoolManager(driver_base.BasePoolManager):

    def create(self, context, pool):
            super(PoolManager, self).delete(context, pool)
            # 从数据库中选择了一个agent
            agent = self.driver.get_loadbalancer_agent(
                context, pool.listener.loadbalancer.id)
            # 向这个agent发送create_pool这个任务
            # 进入LoadBalancerAgentApi的create_pool
            self.driver.agent_rpc.create_pool(context, pool, agent['host'])

class LoadBalancerAgentApi(object):
    """Plugin side of plugin to agent RPC API."""

    def __init__(self, topic):
        target = messaging.Target(topic=topic, version='1.0')
        self.client = n_rpc.get_client(target,
                                       serializer=DataModelSerializer())

    def create_pool(self, context, pool, host):
            cctxt = self.client.prepare(server=host)
            # 向agent发送create_pool的消息
            cctxt.cast(context, 'create_pool', pool=pool)
```
{% endcode %}

## agent（driver）部分

查看入口函数，可见 LbaasAgentManager 为 api 的 manager

**agent 的入口**

{% code title="neutron_lbaas.agent.agent.py - main" %}
```python
def main():
    # mgr指向LbaasAgentManager
    # 创建pool即为create_pool函数
    mgr = manager.LbaasAgentManager(cfg.CONF)
    svc = LbaasAgentService(
        host=cfg.CONF.host,
        topic=constants.LOADBALANCER_AGENTV2,
        manager=mgr
    )
    service.launch(cfg.CONF, svc).wait()

```
{% endcode %}

### **agent 处理 plugin 下发的消息**

{% code title="neutron_lbaas.agent.agent_manager.py" %}
```python
class LbaasAgentManager(periodic_task.PeriodicTasks):

    def create_pool(self, context, pool):

        pool = data_models.Pool.from_dict(pool)
        # 根据pool对象的listener.loadbalancer.id获取driver
        # 根据配置文件找到device_driver参数的值
        driver = self._get_driver(pool.listener.loadbalancer.id)
        try:
            # driver创建pool
            # 如选用默认的haproxy，函数路径如下：
            # PoolManager中的create函数
            driver.pool.create(pool)
        except Exception:
            self._handle_failed_driver_call('create', pool, driver.get_name())
        else:
            # 回报状态
            self._update_statuses(pool)
```
{% endcode %}

### **agent 将任务下发给 driver**

{% code title="neutron_lbaas.drivers.haproxy.namespace_driver.py" %}
```python
NS_PREFIX = 'qlbaas-'

def get_ns_name(namespace_id):
    # 返回namespace的名字
    return NS_PREFIX + namespace_id

class HaproxyNSDriver(agent_device_driver.AgentDeviceDriver):

    def __init__(self, conf, plugin_rpc):

        super(HaproxyNSDriver, self).__init__(conf, plugin_rpc)
        # 指向了PoolManager
        # self（也就是PoolManager中的self.driver）即为HaproxyNSDriver
        self._pool = PoolManager(self)
        # 指向了LoadBalancerManager
        # self（也就是LoadBalancerManager中的self.driver）即为HaproxyNSDriver
        self._loadbalancer = LoadBalancerManager(self)

    @property
    def loadbalancer(self):
        return self._loadbalancer

    @property
    def pool(self):
        return self._pool
```
{% endcode %}

{% code title="neutron_lbaas.drivers.haproxy.namespace_driver.py" %}
```python
class PoolManager(agent_device_driver.BasePoolManager):
    def create(self, pool):
        # 调用了LoadBalancerManager的refresh函数
        self.driver.loadbalancer.refresh(pool.listener.loadbalancer)

class LoadBalancerManager(agent_device_driver.BaseLoadBalancerManager):

    def refresh(self, loadbalancer):
        # 调用了LbaasAgentApi的get_loadbalancer函数
        loadbalancer_dict = self.driver.plugin_rpc.get_loadbalancer(
            loadbalancer.id)
        # 根据返回的lb的dict生成一个lb的object
        loadbalancer = data_models.LoadBalancer.from_dict(loadbalancer_dict)
        # 部署lb
        # 调用了HaproxyNSDriver的deploy_instance函数
        if (not self.driver.deploy_instance(loadbalancer) and
                self.driver.exists(loadbalancer.id)):
            self.driver.undeploy_instance(loadbalancer.id)
```
{% endcode %}

### **driver 通过 rpc 访问 plugin 侧的 db**

{% code title="neutron_lbaas.agent.agent_api.py" %}
```python
class LbaasAgentApi(object):
    def get_loadbalancer(self, loadbalancer_id):
        cctxt = self.client.prepare()
        # 调用了LoadBalancerCallbacks的get_loadbalancer函数
        # 返回一个loadbalancer的dict
        # 回到LoadBalancerManager的refresh函数
        return cctxt.call(self.context, 'get_loadbalancer',
                          loadbalancer_id=loadbalancer_id)

```
{% endcode %}

{% code title="neutron_lbaas.drivers.common.agent_callbacks.py" %}
```python
class LoadBalancerCallbacks(object):
    def get_loadbalancer(self, context, loadbalancer_id=None):
        # 这里指向LoadBalancerPluginDbv2的get_loadbalancer函数
        lb_model = self.plugin.db.get_loadbalancer(context, loadbalancer_id)
        if lb_model.vip_port and lb_model.vip_port.fixed_ips:
            for fixed_ip in lb_model.vip_port.fixed_ips:
                subnet_dict = self.plugin.db._core_plugin.get_subnet(
                    context, fixed_ip.subnet_id
                )
                setattr(fixed_ip, 'subnet', data_models.Subnet.from_dict(
                    subnet_dict))
        if lb_model.provider:
            device_driver = self.plugin.drivers[
                lb_model.provider.provider_name].device_driver
            setattr(lb_model.provider, 'device_driver', device_driver)
        # 将这个LoadBalancer的object转换为dict
        lb_dict = lb_model.to_dict(stats=False)

        # 返回这个字典
        # 回到LbaasAgentApi的get_loadbalancer函数
        return lb_dict
```
{% endcode %}

{% code title="neutron_lbaas.db.loadbalancer.loadbalancer_dbv2.py" %}
```python
class LoadBalancerPluginDbv2(base_db.CommonDbMixin,
                             agent_scheduler.LbaasAgentSchedulerDbMixin):
    def get_loadbalancer(self, context, id):
        lb_db = self._get_resource(context, models.LoadBalancer, id)
        # 返回一个LoadBalancer的object
        # 回到LoadBalancerCallbacks的get_loadbalancer函数
        return data_models.LoadBalancer.from_sqlalchemy_model(lb_db)
```
{% endcode %}

### **driver 开始执行实际操作**

{% code title="neutron_lbaas.drivers.haproxy.namespace_driver.py" %}
```python
class HaproxyNSDriver(agent_device_driver.AgentDeviceDriver):

    @n_utils.synchronized('haproxy-driver')
    def deploy_instance(self, loadbalancer):
        # 部署一个lb，如果已经有，则返回True，否则返回False
        # deployable函数用于检查是否可以部署
        # 如listener是否已经准备就绪或可用，如可以返回True，不可以则返回False
        if not self.deployable(loadbalancer):
            LOG.info(_LI("Loadbalancer %s is not deployable.") %
                     loadbalancer.id)
            return False

        if self.exists(loadbalancer.id):
            self.update(loadbalancer)
        else:
            # 创建，进入create函数
            self.create(loadbalancer)
        return True

    def deployable(self, loadbalancer):
        # 如果lb已经active了，返回True，否则返回False
        if not loadbalancer:
            return False
        acceptable_listeners = [
            listener for listener in loadbalancer.listeners
            if (listener.provisioning_status != constants.PENDING_DELETE and
                listener.admin_state_up)]
        return (bool(acceptable_listeners) and loadbalancer.admin_state_up and
                loadbalancer.provisioning_status != constants.PENDING_DELETE)
        # 回到HaproxyNSDriver的deploy_instance函数

    def create(self, loadbalancer):
        namespace = get_ns_name(loadbalancer.id)
        # 挂载网卡，将进入_plug函数
        self._plug(namespace, loadbalancer.vip_port, loadbalancer.vip_address)
        # 孵化lb，进入_spawn函数
        self._spawn(loadbalancer) 
```
{% endcode %}

#### `_plug`

{% code title="neutron_lbaas.drivers.haproxy.namespace_driver.py" %}
```python
def _plug(self, namespace, port, vip_address, reuse_existing=True):
        # 调用LbaasAgentApi的plug_vip_port函数，rpc的publisher
        self.plugin_rpc.plug_vip_port(port.id)

        # tap××××××××-××
        interface_name = self.vif_driver.get_device_name(port)

        # 如果这个namespace中有这个interface，返回True，否则返回False
        if ip_lib.device_exists(interface_name,
                                namespace=namespace):
            if not reuse_existing:
                raise exceptions.PreexistingDeviceFailure(
                    dev_name=interface_name
                )
        else:
            self.vif_driver.plug(
                port.network_id,
                port.id,
                interface_name,
                port.mac_address,
                namespace=namespace
            )

        # 根据port的信息建立l3
        self.vif_driver.init_l3(interface_name, cidrs, namespace=namespace)

        # Haproxy socket binding to IPv6 VIP address will fail if this address
        # is not yet ready(i.e tentative address).
        if netaddr.IPAddress(vip_address).version == 6:
            device = ip_lib.IPDevice(interface_name, namespace=namespace)
            device.addr.wait_until_address_ready(vip_address)

        gw_ip = port.fixed_ips[0].subnet.gateway_ip

        if not gw_ip:
            host_routes = port.fixed_ips[0].subnet.host_routes
            for host_route in host_routes:
                if host_route.destination == "0.0.0.0/0":
                    gw_ip = host_route.nexthop
                    break
        else:
        cmd = ['route', 'add', 'default', 'gw', gw_ip]
        # 添加默认路由
        ip_wrapper = ip_lib.IPWrapper(namespace=namespace)
        # ip netns exec ns env *** route add default gw gw_ip
        ip_wrapper.netns.execute(cmd, check_exit_code=False)
        # When delete and re-add the same vip, we need to
        # send gratuitous ARP to flush the ARP cache in the Router.
        gratuitous_arp = self.conf.haproxy.send_gratuitous_arp
        if gratuitous_arp > 0:
            for ip in port.fixed_ips:
                cmd_arping = ['arping', '-U',
                              '-I', interface_name,
                              '-c', gratuitous_arp,
                              ip.ip_address]
                # ip netns exec ns env *** arping \
                # -U -I interface_name -c gratuitous_arp ip.ip_address
                ip_wrapper.netns.execute(cmd_arping, check_exit_code=False)
        # 回到create函数
```
{% endcode %}

#### `_spawn`

{% code title="neutron_lbaas.drivers.haproxy.namespace_driver.py" %}
```python
def _spawn(self, loadbalancer, extra_cmd_args=()):
        namespace = get_ns_name(loadbalancer.id)
        conf_path = self._get_state_file_path(loadbalancer.id, 'haproxy.conf')
        pid_path = self._get_state_file_path(loadbalancer.id,
                                             'haproxy.pid')
        sock_path = self._get_state_file_path(loadbalancer.id,
                                              'haproxy_stats.sock')
        user_group = self.conf.haproxy.user_group
        haproxy_base_dir = self._get_state_file_path(loadbalancer.id, '')
        jinja_cfg.save_config(conf_path,
                              loadbalancer,
                              sock_path,
                              user_group,
                              haproxy_base_dir)
        cmd = ['haproxy', '-f', conf_path, '-p', pid_path]
        cmd.extend(extra_cmd_args)

        ns = ip_lib.IPWrapper(namespace=namespace)
        # ip netns exec ns env *** haproxy -f conf_path -p pid_path
        ns.netns.execute(cmd)

        # remember deployed loadbalancer id
        self.deployed_loadbalancers[loadbalancer.id] = loadbalancer
```
{% endcode %}



## 操作步骤（未实践）

1 如使用 rdo 安装，需要设置 neutron 的 lbaas 地址，`--neutron-lbaas-hosts`

2 迁移 lbaas 的数据库至 neutron 数据库

```python
$ neutron-db-manage --service lbaas upgrade head
```

3 创建一个用于提供 vip 的 subnet，此处命名 private-subnet

4 创建 loadbalancer，此处命名 test-lb

```python
$ neutron lbaas-loadbalancer-create --name test-lb private-subnet
```

5 创建 listener，此处命名 test-lb-http

```python
$ neutron lbaas-listener-create \
  --name test-lb-http \
  --loadbalancer test-lb \
  --protocol HTTP \
  --protocol-port 80
```

6 创建 pool，此处命名 test-lb-pool-http

```python
$ neutron lbaas-pool-create \
  --name test-lb-pool-http \
  --lb-algorithm ROUND_ROBIN \
  --listener test-lb-http \
  --protocol HTTP
```

7 创建 member

```python
$ neutron lbaas-member-create \
  --subnet private-subnet \
  --address 192.168.1.16 \
  --protocol-port 80 \
  test-lb-pool-http
$ neutron lbaas-member-create \
  --subnet private-subnet \
  --address 192.168.1.17 \
  --protocol-port 80 \
  test-lb-pool-http
```

8 创建 healthmonitor

```python
$ neutron lbaas-healthmonitor-create \
  --delay 5 \
  --max-retries 2 \
  --timeout 10 \
  --type HTTP \
  --pool test-lb-pool-http
```

## References

* 原文 [Liberty 版本 Neutron LBaas 学习](https://www.laminar.fun/Openstack/2016-09-14-neutron-lbaas-arch-code-operation.html)
