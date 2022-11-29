---
title: Mptcp linux kernel implementation
date: 2022-11-28 11:19:15
tags: 
- multipath
- mptcp
- linux
categories: 
- network
---

最近做一些多路径相关研究，主要有mptcp和mpquic两种方式，多路径最终在于解决3个问题：**多条路径如何建立**、**多条路径如何分配数据包**以及**多条路径如何做好拥塞控制**。

<!--more-->

![image-20210817150015171](https://cdn.jsdelivr.net/gh/xiaohuiluo/images/img/image-20210817150015171.png)

这里介绍一下mptcp，mptcp linux内核实现针对多路径问题分为如下三个核心部分，使用mptcp时根据不同使用场景对其进行不同的配置是很有必要的。

> 1. path manager : 路径管理
> 2. scheduler control : 调度控制算法
> 3. congestion control : 拥塞控制算法

## path manager

### path manager分析

Linux最新mptcp_v0.95实现中一共包含如下4种路径管理(subflow)机制,还有一种默认dummy(啥都不做)路径管理。

> 1. Full-Mesh
> 2. ndiff-ports
> 3. Binder
> 4. Netlink
> 5. dummy(默认)

```bash
config MPTCP_FULLMESH
	tristate "MPTCP Full-Mesh Path-Manager"
	depends on MPTCP=y
	---help---
	  This path-management module will create a full-mesh among all IP-addresses.

config MPTCP_NDIFFPORTS
	tristate "MPTCP ndiff-ports"
	depends on MPTCP=y
	---help---
	  This path-management module will create multiple subflows between the same
	  pair of IP-addresses, modifying the source-port. You can set the number
	  of subflows via the mptcp_ndiffports-sysctl.

config MPTCP_BINDER
	tristate "MPTCP Binder"
	depends on (MPTCP=y)
	---help---
	  This path-management module works like ndiffports, and adds the sysctl
	  option to set the gateway (and/or path to) per each additional subflow
	  via Loose Source Routing (IPv4 only).

config MPTCP_NETLINK
	tristate "MPTCP Netlink Path-Manager"
	depends on MPTCP=y
	---help---
	  This path-management module is controlled over a Netlink interface. A userspace
	  module can therefore control the establishment of new subflows and the policy
	  to apply over those new subflows for every connection.
```

### path manager配置

![image-20210812102200625](https://cdn.jsdelivr.net/gh/xiaohuiluo/images/img/image-20210812102200625.png)

`sysctl net.mptcp.mptcp_path_manager=default/fullmesh/ndiffports/binder/netlink`

## scheduler control

### scheduler分析

Linux最新mptcp_v0.95实现中一共包含如下4种多路径调度算法，默认使用minRTT调度算法。

> 1. minRTT(默认)
> 2. Round-Robin
> 3. Redundant
> 4. BLEST(实验阶段)

```bash
	config DEFAULT_SCHEDULER
		bool "Default"
		---help---
		  This is the default scheduler, sending first on the subflow
		  with the lowest RTT.

	config DEFAULT_ROUNDROBIN
		bool "Round-Robin" if MPTCP_ROUNDROBIN=y
		---help---
		  This is the round-rob scheduler, sending in a round-robin
		  fashion..

	config DEFAULT_REDUNDANT
		bool "Redundant" if MPTCP_REDUNDANT=y
		---help---
		  This is the redundant scheduler, sending packets redundantly over
		  all the subflows.
	config MPTCP_BLEST
		tristate "MPTCP BLEST"
		depends on MPTCP=y
		---help---
	  	  This is an experimental BLocking ESTimation-based (BLEST) scheduler.

```

### scheduler配置

![image-20210811151032091](https://cdn.jsdelivr.net/gh/xiaohuiluo/images/img/image-20210811151032091.png)

`sysctl net.mptcp.mptcp_scheduler=default/roundrobin/redundant/blest`

[`BLEST` https://ieeexplore.ieee.org/document/7497206](https://ieeexplore.ieee.org/document/7497206)

## congestion control

### congestion control分析

Linux最新mptcp_v0.95实现中一共包含如下5种mptcp拥塞算法，还有很多种tcp拥塞算法，默认使用cubic算法，安装github生成安装包及使用mptcp内核并不会默认使用mptcp相关拥塞算法，需要自己加载进内核并启用mptcp拥塞算法，有的新版本支持特性可能需要自己配置linux内核并重新编译。

> 1. LIA
> 2. OLIA
> 3. WVEGAS
> 4. BALIA
> 5. MCTCPDESYNC

```bash
config TCP_CONG_LIA
	tristate "MPTCP Linked Increase"
	depends on MPTCP
	default n
	---help---
	MultiPath TCP Linked Increase Congestion Control
	To enable it, just put 'lia' in tcp_congestion_control

config TCP_CONG_OLIA
	tristate "MPTCP Opportunistic Linked Increase"
	depends on MPTCP
	default n
	---help---
	MultiPath TCP Opportunistic Linked Increase Congestion Control
	To enable it, just put 'olia' in tcp_congestion_control

config TCP_CONG_WVEGAS
	tristate "MPTCP WVEGAS CONGESTION CONTROL"
	depends on MPTCP
	default n
	---help---
	wVegas congestion control for MPTCP
	To enable it, just put 'wvegas' in tcp_congestion_control

config TCP_CONG_BALIA
	tristate "MPTCP BALIA CONGESTION CONTROL"
	depends on MPTCP
	default n
	---help---
	Multipath TCP Balanced Linked Adaptation Congestion Control
	To enable it, just put 'balia' in tcp_congestion_control

config TCP_CONG_MCTCPDESYNC
	tristate "DESYNCHRONIZED MCTCP CONGESTION CONTROL (EXPERIMENTAL)"
	depends on MPTCP
	default n
	---help---
	Desynchronized MultiChannel TCP Congestion Control. This is experimental
	code that only supports single path and must have set mptcp_ndiffports
	larger than one.
	To enable it, just put 'mctcpdesync' in tcp_congestion_control
	For further details see:
	  http://ieeexplore.ieee.org/abstract/document/6911722/
	  https://doi.org/10.1016/j.comcom.2015.07.010
```

### congestion control配置

![image-20210811150851262](https://cdn.jsdelivr.net/gh/xiaohuiluo/images/img/image-20210811150851262.png)

`sysctl net.ipv4.tcp_congestion_control=lia/olia/wvegas/balia/mctcpdesync`

[`lia` https://datatracker.ietf.org/doc/rfc6356/](https://datatracker.ietf.org/doc/rfc6356/)
[`olia` https://datatracker.ietf.org/doc/draft-khalili-mptcp-congestion-control/](https://datatracker.ietf.org/doc/draft-khalili-mptcp-congestion-control/)
[`wvegas` http://tools.ietf.org/html/draft-xu-mptcp-congestion-control/](http://tools.ietf.org/html/draft-xu-mptcp-congestion-control/)
[`balia` https://datatracker.ietf.org/doc/draft-walid-mptcp-congestion-control/](https://datatracker.ietf.org/doc/draft-walid-mptcp-congestion-control/)
[`DMCTCP` https://ieeexplore.ieee.org/abstract/document/6911722/](https://ieeexplore.ieee.org/abstract/document/6911722/)

