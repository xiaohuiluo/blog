---
title: onos-netcfg
date: 2022-12-29 17:24:11
categories: 
	- network
tags: 
	- onos
	- sdn
	- netcfg
---

本篇介绍NetCfg网络配置子系统核心功能及实现及ONOS中原生App中如何使用NetCfg进行动态配置。

<!--more-->

## NetCfg子系统

![netcfg subsystem](https://gitee.com/martrix/blog-images/raw/master/img/20221229-154514.png)

Network Cfg(NetCfg) SubSystem主要用于对子系统进行配置，我们也可在ONOS App开发中使用(对App进行相应的动态配置)，Netcfg主要实现如上图所示，核心实现机制如下：

* Netcfg核心在于实现了使用Listener-Service机制，并提供Factory机制来扩展新的动态配置支持，基于NetworkConfigStore实现配置持久化
* 用户基于Factory机制可扩展注册新的动态配置类如DemoConfig
* 用户可扩展增加监听类如MyConfigListener注册监听来获取动态配置Config状态变化Event，并在其内部根据Config是否为DemoConfig或其他已有配置来执行相应配置动作
* ONOS其他子系统和App中大量使用NetCfg机制来动态配置，如Device，Host等

## dhcp中的NetCfg

ONOS自带App dhcp(apps/dhcp)中使用netcfg进行dhcp的动态配置代码，若我们基于maven或onos中使用bazel扩展配置也是按同样步骤进行，动态配置在于将json配置文件信息映射到java的内存中，netcfg支持了json中单个对象及数组的配置。

### dhcp中NetCfg代码分析

1. netcfg服务集成

```java
// cfg service服务集成    
@Reference(cardinality = ReferenceCardinality.MANDATORY)
protected NetworkConfigRegistry cfgService;
```

2. factories定义新的配置类(DhcpConfig)

```java
// factories工厂支持新的配置类，可以支持object和list配置，通过ConfigFactory两个构造函数来区分
private final Set<ConfigFactory> factories = ImmutableSet.of(
            new ConfigFactory<ApplicationId, DhcpConfig>(APP_SUBJECT_FACTORY,
                    DhcpConfig.class,
                    "dhcp") {
                @Override
                public DhcpConfig createConfig() {
                    return new DhcpConfig();
                }
            }
    );

// 新的DhcpConfig配置类， 抽象类Config中object对应单对象配置，array对应数组配置
public class DhcpConfig extends Config<ApplicationId> {
    //...
}

```

3. cfgListener自定义扩展配置监听类(InternalConfigListener)

```java
// 扩展netcfg的监听类对象
private final InternalConfigListener cfgListener = new InternalConfigListener();

// 自定义扩展配置监听类
private class InternalConfigListener implements NetworkConfigListener {
    
    // ...
    
    @Override
    public void event(NetworkConfigEvent event) {
        // 监听netcfg配置变化，变化为DhcpConfig时进行相应的动态操作
        if ((event.type() == NetworkConfigEvent.Type.CONFIG_ADDED ||
             event.type() == NetworkConfigEvent.Type.CONFIG_UPDATED) &&
            event.configClass().equals(DhcpConfig.class)) {

            DhcpConfig cfg = cfgService.getConfig(appId, DhcpConfig.class);
            reconfigureNetwork(cfg);
            log.info("Reconfigured");
        }
    }
}

```

4. 自定义factories的注册&注销和自定义listener的添加&删除

```java
// dhcp activate
// 自定义扩展listener添加
cfgService.addListener(cfgListener);
// 自定义扩展factories注册
factories.forEach(cfgService::registerConfigFactory);

// dhcp deactivate
cfgService.removeListener(cfgListener);
factories.forEach(cfgService::unregisterConfigFactory);

```

### dhcp配置文件

在onos/tools/test/configs/office-dhcp.json中是dhcp动态配置样例配置文件，用户可通过onos-netcfg命令或北向api接口将配置文件配置到onos dhcp中。

```json
{
  "apps": {
    "org.onosproject.dhcp" : {
      "dhcp" : {
        "ip": "10.1.11.50",
        "mac": "ca:fe:ca:fe:ca:fe",
        "subnet": "255.255.252.0",
        "broadcast": "10.1.11.255",
        "router": "10.1.8.1",
        "domain": "8.8.8.8",
        "ttl": "63",
        "lease": "300",
        "renew": "150",
        "rebind": "200",
        "delay": "2",
        "timeout": "150",
        "startip": "10.1.11.51",
        "endip": "10.1.11.100"
      }
    }
  }
}
```

### dhcp动态配置

本地部署好onos且启用好dhcp App后可通过onos-netcfg命令(若已安装好onos开发环境)进行dhcp动态配置，本地或非本地也可通过北向api接口进行动态配置。

```bash
# onos-netcfg本地动态配置dhcp
onos-netcfg localhost onos/tools/test/configs/office-dhcp.json 

# 使用北向api动态配置dhcp
curl --fail -sSL --user onos:rocks --noproxy localhost -X POST -H 'Content-Type:application/json' \
		http://localhost:8181/onos/v1/network/configuration -d @onos/tools/test/configs/office-dhcp.json
```

配置后可在onos的日志中看到动态配置的日志输出

```
onos_1  | 09:13:23.102 INFO  [ApplicationManager] Application org.onosproject.dhcp has been activated
onos_1  | 09:13:23.306 INFO  [DhcpManager] Host discovery is set to false
onos_1  | 09:13:23.306 INFO  [DhcpManager] Modified
onos_1  | 09:14:05.588 INFO  [DhcpManager] Reconfigured

```

