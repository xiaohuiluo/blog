---
title: ONOS Subsystem
date: 2022-12-29 15:46:45
categories: 
	- network
tags: 
	- onos
	- sdn
	- subsystem
---

## ONOS SubSystems

ONOS核心由各个子系统构成，这些子系统一方面提供网络操作系统核心功能，一方面为用户扩展开发提供支持。

<!--more-->

![image-20221216105907047](https://gitee.com/martrix/blog-images/raw/master/img/20221216-105911.png)

ONOS子系统包含网络相关的设备，主机，网络业务，网络转发配置(流表)，设备驱动，网络动态配置，底层系统相关分布式节点Mastership，分布式存储Storage及GUI，主要的核心子系统如下：

* Topology, Device, Link, Host
* Intent, Path, Tunnel, Packet
* Flow Objective, Group, Meter, Flow Rule
* Driver, Network Cfg, Storage, Mastership
* UI Extension

## SubSystem Structure

ONOS大部分子系统都集成或部分集成Listener-Service和Provider-Registry两大机制，并且基于ONOS中Store做分布式存储持久化，子系统核心结构如下：

![subsystem structure](https://gitee.com/martrix/blog-images/raw/master/img/20221216-110923.png)

1. 北向SubSystem通过Listener-Service机制实现和App解耦
2. 南向SubSystem通过Provider-Registry机制实现和南向协议解耦(部分子系统不涉就不集成该机制)
3. SubSystem持久化通过ONOS Store(Storage)机制实现分布式存储

![subsystem components](https://gitee.com/martrix/blog-images/raw/master/img/20221216-111601.png)

ONOS 子系统组成：

1. **Listener-Service**机制：Service监听SubSystem状态变化通过Event通知Listener(**可扩展**)执行相应操作
2. **Provider-Registry**机制：支持多个Provider(**可扩展**)注册到Registry对SubSystem状态进行更新
3. SubSystemStore：持久化SubSystem状态数据(分布式存储，数据持久化，增删改查)
4. SubSystemNetCfg：支持对SubSystem的动态配置

后面会陆续更新ONOS中主要核心子系统的分析，大部分子系统代码实现上是相似的。

