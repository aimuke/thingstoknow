# OpenStack Neutron LoadBalance源码解析（一）

作者：林凯

团队：华为杭州OpenStack团队

在OpenStackGrizzly版本中，Neutron（当时叫Quantum）组件引入了一个新的网络服务：LoadBalance（LBaaS），关于LoadBalance的框架和基础知识方面网上已经有了一些好文章，在此不再赘述。本文将对LoadBalancer的代码流程及实现进行初步解析，肯定会有错误和不严谨的地方，需要大家给予指正。

推荐一些基础知识的文章给大家，大家看完之后再看源码会更加简单一些。

[http://blog.csdn.net/quqi99/article/details/9898139](http://blog.csdn.net/quqi99/article/details/9898139)

[http://www.cnblogs.com/feisky/p/3853734.html](http://www.cnblogs.com/feisky/p/3853734.html)

[http://www.ustack.com/blog/neutron\_loadbalance/](http://www.ustack.com/blog/neutron_loadbalance/)

Neutron LoadBalancer的基本使用步骤是：

1\)        租户创建一个pool，初始时的member个数为0；

2\)        租户在该pool内创建一个或多个 member

3\)        租户创建一个或多个 health monitor

4\)        租户将health monitors与pool关联

5\)        租户使用pool创建vip

## 创建流程

首先我们来看下我们在创建pool、member、healthmonitor和vip的时候，代码都做了哪些事情。

![](../.gitbook/assets/image%20%2846%29.png)

图1  LoadBalance整体流程框架

## plugin create\_pool

从上图可以看到租户创建pool等的请求首先发送到LBaaSPlugin进行处理，所以我们对代码的解析同样从这开始。在/neutron/services/loadbalancer/plugin.py中我们可以看到对应的create\_pool等方法：

```python
# pool作为LB v1的根对象，是工作流的起点
    def create_pool(self, context, pool):
        provider_name =self._get_provider_name(context, pool['pool'])
        # DB中创建pool对象
        p = super(LoadBalancerPlugin,self).create_pool(context, pool)

       self.service_type_manager.add_resource_association(
            context,
            constants.LOADBALANCER,
            provider_name, p['id'])
        #need to add provider name to pooldict,
        #because provider was not known to dbplugin at pool creation
        p['provider'] = provider_name
        driver = self.drivers[provider_name]
        try:
            # 调用默认provider中的驱动创建pool
            driver.create_pool(context, p)
        except loadbalancer.NoEligibleBackend:
            # that should catch cases whenbackend of any kind
            # is not available (agent,appliance, etc)
            self.update_status(context,ldb.Pool,
                               p['id'], constants.ERROR,
                               "Noeligible backend")
            raiseloadbalancer.NoEligibleBackend(pool_id=p['id'])
        return p
```

## driver create\_pool

其中driver.create\_pool\(context,p\)是关键方法

driver =self.drivers\[provider\_name\]，可知：根据配置文件中的provider设置，调用相应的driver。

在neutron.conf中可以看到默认的设置为：service\_provider=LOADBALANCER:Haproxy:neutron.services.loadbalancer.drivers.haproxy.plugin\_driver.HaproxyOnHostPluginDriver:default

所以跳转至/neutron/services/loadbalancer/haproxy/plugin\_driver.py中：

```python
class HaproxyOnHostPluginDriver(agent_driver_base.AgentDriverBase):
    device_driver =namespace_driver.DRIVER_NAME
```

这个类中并无create\_pool的方法，所以去他的父类AgentDriverBase中查看：

```python
def create_pool(self, context, pool):
        # 首先通过agent scheduler分配agent
        agent =self.pool_scheduler.schedule(self.plugin, context, pool,
                                            self.device_driver)
        if not agent:
            raiselbaas_agentscheduler.NoEligibleLbaasAgent(pool_id=pool['id'])
        self.agent_rpc.create_pool(context,pool, agent['host'],
                                  self.device_driver)
```

## agent scheduler分配agent

在这里首先通过agent scheduler分配agent，看下具体的实现方式：

```python
def schedule(self, plugin, context, pool, device_driver):
        # 为pool分配一个active的loadbalanceragent，如果其上没有已经使能的agent
        with context.session.begin(subtransactions=True):
            lbaas_agent =plugin.get_lbaas_agent_hosting_pool(
                context, pool['id'])
            if lbaas_agent:
                LOG.debug(_('Pool %(pool_id)shas already been hosted'
                            ' by lbaas agent%(agent_id)s'),
                          {'pool_id':pool['id'],
                           'agent_id':lbaas_agent['id']})
                return
            # 获取active的agent
            active_agents =plugin.get_lbaas_agents(context, active=True)
            if not active_agents:
                LOG.warn(_('No active lbaasagents for pool %s'), pool['id'])
                return

            # 根据device_driver筛选候选的agent
            candidates =plugin.get_lbaas_agent_candidates(device_driver,
                                                          active_agents)
            if not candidates:
                LOG.warn(_('No lbaas agentsupporting device driver %s'),
                         device_driver)
                return

            # 随机选取一个合适的候选agent
            chosen_agent =random.choice(candidates)
            # 与pool进行绑定
            binding =PoolLoadbalancerAgentBinding()
            binding.agent = chosen_agent
            binding.pool_id = pool['id']
            context.session.add(binding)
            LOG.debug(_('Pool %(pool_id)s isscheduled to '
                        'lbaas agent%(agent_id)s'),
                      {'pool_id': pool['id'],
                       'agent_id':chosen_agent['id']})
            return chosen_agent
```

我们刚才在Haproxy的plugindriver中，可知:

```python
device_driver = namespace_driver.DRIVER_NAME
```

## agent\_manager 接收到请求

所以根据device\_driver选择的agent是Haproxy的namespace\_driver，通过agent将创建pool的请求转发给device\_driver\(HaproxyNSDriver\)这个过程并不是直接就一步到位的，其中agent发送RPC异步请求，此时agent\_manager接收到请求

```python
def create_pool(self, context, pool, driver_name):
        if driver_name not inself.device_drivers:
            LOG.error(_('No device driver onagent: %s.'), driver_name)
           self.plugin_rpc.update_status('pool', pool['id'], constants.ERROR)
            return
        # 获取相应的driver（根据设置默认为HaproxyNSDriver）
        driver =self.device_drivers[driver_name]
        try:
            driver.create_pool(pool)
        except Exception:
           self._handle_failed_driver_call('create', 'pool', pool['id'],
                                            driver.get_name())
        else:
            self.instance_mapping[pool['id']] =driver_name
            # 之后更新数据库中的状态
           self.plugin_rpc.update_status('pool', pool['id'], constants.ACTIVE)
```

## HaproxyNSDriver create\_pool

可知这里调用HaproxyNSDriver中的create\_pool函数，在Haproxy的namespace\_driver.py中找到对应的方法：

```python
def create_pool(self, pool):
        # nothing to do here because a poolneeds a vip to be useful
        # 当没有vip的情况下，不做操作。
        Pass
```

而member，healthmonitor和vip的创建流程与之类似，可以参照这个流程进行解析。

至此，Neutron LoadBalance的源码解析第一部分内容就结束了，下一部分内容敬请期待。

**时间：** 12-23

## References

* 原文 [OpenStack Neutron LoadBalance源码解析（一）](https://www.biecuoliao.com/pa/4pOEvK3.html)

