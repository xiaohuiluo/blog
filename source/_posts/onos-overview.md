---
title: ONOS Overview
date: 2022-12-13 10:30:09
categories: 
	- network
tags: 
	- onos
	- sdn
---

Open Network Operating System (ONOS®) 是下一代SDN/NFV的领先的开源SDN控制器之一。

<!--more-->

目前开源主流SDN控制器主要有ODL(OpenDayLight)和ONOS两大平台，其他的如Ryu、Floodlight等逐渐不再进行更新维护了，ODL和ONOS都基于Java的OSGi架构开发，所以两者在设计理念和开发上基本一致，熟悉其中一个后学习另一个也很容易上手，ONOS由ONF基金会主导，ODL由Linux基金会主导。

## ONOS架构

<img src="https://gitee.com/martrix/blog-images/raw/master/img/20221212-170148.png" alt="onos arth" style="zoom:80%;" />

ONOS整体架构基于OSGi(Apache Karaf实现)之上，核心提供多个网络相关的核心子系统，南向通过丰富的协议及网元设备行为驱动机制来支撑多厂商多类型设备支持，北向通过北向接口开放网络功能，值得提的是ODL的设计理念和ONOS类似。

架构特点：

- High-availability, scalability and performance(高可用、可扩展伸缩、高性能)
- Strong abstractions and simplicity to develop apps and solutions(高度抽象及简化apps和方案开发)
- Protocol and device behaviour independence(南向协议及独立设备行为驱动机制-避免厂商锁定)
- Separation of concerns and modularity(解耦及模块化理念)

## ONOS OSGi

ONOS底层采用Java OSGi架构(Apache Karaf)来构建平台，ODL同样采用了该架构。

![OSGi](https://gitee.com/martrix/blog-images/raw/master/img/20221212-173057.png)

OSGi架构特点：

- Bundles – Bundles are the OSGi components made by the developers.
- Services – The services layer connects bundles in a dynamic way by offering a publish-find-bind model for plain old Java objects.
- Life-Cycle – The API to install, start, stop, update, and uninstall bundles.
- Modules – The layer that defines how a bundle can import and export code.
- Security – The layer that handles the security aspects.
- Execution Environment – Defines what methods and classes are available in a specific platform.

![Apache Karaf](https://gitee.com/martrix/blog-images/raw/master/img/20221213-085554.png)

Apache Karaf架构实现特点：

- Hot deployment(热部署)
- Dynamic configuration(动态配置)
- Logging system(日志系统)
- Provisioning(应用提供)
- Shell Console(Shell命令行控制台)
- Remote management(远程管理)
- WebConsole(Web控制台)
- Security(安全)
- Instances management(实例管理)
- Docker & Cloud ready(云化容器化支持)

OSGi(Apache Karaf实现)架构模块化开发、动态配置‘、热部署及完善的web应用平台等特性契合了电信级软件的要求。

## ONOS子系统

ONOS子系统提供最核心的网络相关功能，这些功能为上层网络应用开发提供底层接口，子系统设计和实现上采用了相同的理念。

![onos subsystem](https://gitee.com/martrix/blog-images/raw/master/img/20221212-172008.png)

如上红色框部分就是ONOS实现的核心子系统，包含Topology网络拓扑、Link网络连接、Network Cfg网络配置、Flow Rule/Objective流表、Device网络设备、Host网络主机，Packet网络数据包等，子系统大部分采用**Provider-Registry**机制和**Service-Listerner**机制设计理念来达到模块化解耦的目的。

![onos subsystem provider&listener](https://gitee.com/martrix/blog-images/raw/master/img/20221212-172853.png)

## ONOS编译开发工具

- 编译系统演进(maven --> buck --> **bazel**)，bazel提供大工程快速编译能力

- 自带开发工具

  ![onos dev tools](https://gitee.com/martrix/blog-images/raw/master/img/20221213-093135.png)

## ONOS代码结构

![onos code](https://gitee.com/martrix/blog-images/raw/master/img/20221213-093452.png)

## ONOS App

ONOS上层网络功能可通过开发ONOS支持的App应用来支持，ONOS内部也有实现丰富的网络App样例(简单的如fwd，dhcp等、复杂的如Trellis，SONA，OpenCORD等解决方案)，用户可直接在onos代码内使用bazel或者外部使用maven开发网络App应用。

### bazel创建App

- bazel开发app需要在onos代码统一进行编译

- 新app需要加入到onos项目module

- bazel build遵循onos要求

  ![image-20221213094409217](https://gitee.com/martrix/blog-images/raw/master/img/20221213-094409.png)

### maven创建app

```bash
# onos支持版本仓库查询
# https://repo.maven.apache.org/maven2/org/onosproject/onos-bundle-archetype/

# 先配置好onos开发环境
# ONOS 
export WORKSPACE="/home/hui/SDN"
export ONOS_ROOT="$WORKSPACE/onos"
source $ONOS_ROOT/tools/dev/bash_profile

export ONOS_POM_VERSION=2.1.0
# 交互式创建 （首次执行会从maven仓库下载较多依赖库）
onos-create-app
# 参数式创建
onos-create-app onos-app cn.hui onos-app 1.0.0-SNAPSHOT cn.hui

# maven编译下载遇到问题可尝试将maven mirror修改为阿里云mirror等国内mirror
```

![maven app pom](https://gitee.com/martrix/blog-images/raw/master/img/20221213-101837.png)

ONOS App编译生成的.oar文件其实本质上是Karaf中的feature(和ODL中没有本质的区别，因为都是基于Karaf架构)，可以解压.oar看到相应的jar包和feature的xml配置。

## NG-SDN

值得提一下ONF基金会提出下一代SDN，控制面提出μONOS基于微服务控制器，数据面提供stratum+P4作为下一代SDN的技术栈，有兴趣也可以了解。

![NG-SDN](https://gitee.com/martrix/blog-images/raw/master/img/20221213-095944.png)

![μONOS](https://gitee.com/martrix/blog-images/raw/master/img/20221213-100010.png)

###### References

https://wiki.onosproject.org/

https://www.opennetworking.org/onos

https://wiki.onosproject.org/display/ONOS/Downloads

https://www.osgi.org/

http://karaf.apache.org/

https://bazel.build/

https://www.opennetworking.org/cord/

https://wiki.opencord.org/

https://github.com/onosproject

https://www.opennetworking.org/