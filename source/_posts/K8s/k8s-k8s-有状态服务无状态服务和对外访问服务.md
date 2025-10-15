---
title: k8s-有状态服务无状态服务和对外访问服务
categories:
  - k8s
date: 2024-04-12 16:43:25
tags:
  - k8s
cover: /img/k8s.jpg
---
1. 基本概念
无状态服务
无状态服务不会在本地存储持久化数据.多个服务实例对于同一个用户请求的响应结果是完全一致的.这种多服务实例之间是没有依赖关系,比如web应用,在k8s控制器 中动态启停无状态服务的pod并不会对其它的pod产生影响.

有状态服务
有状态服务需要在本地存储持久化数据,典型的是分布式数据库的应用,分布式节点实例之间有依赖的拓扑关系.比如,主从关系. 如果K8S停止分布式集群中任 一实例pod,就可能会导致数据丢失或者集群的crash.

2. Deployment部署的问题?





Deployment被设计用来管理无状态服务的pod,每个pod完全一致.什么意思呢?

无状态服务内的多个Pod创建的顺序是没有顺序的.
无状态服务内的多个Pod的名称是随机的.pod被重新启动调度后,它的名称与IP都会发生变化.
无状态服务内的多个Pod背后是共享存储的.
2.1.新的问题



对于数据库有状态的服务容器编排,Deployment解决方案就变得无能为力了.
比如,Redis是主从的架构,只能允许集群中出现一个主节点提供写,其它节点提供读能力.如果同时出现二个主节点后,必须会出现并发写的 操作,进一步导致集群写数据的不一致.
所以问题来了,针对Redis这种有状态的服务,它管理的多个Pod(代表master/slave角色)必须有自己独立的持久化存储组件.
有状态的服务Pod是用来运行有状态应用的,其在数据卷上存储的数据非常重要,因为Stateful就是要依赖存储数据卷上对每个Pod的状态进行建模与存储.
所以K8S提供了一个新的工具——StatefulSet来统一解决问题.

3. Stateful如何解决问题?
Deployment组件是为无状态服务而设计的,其中的Pod名称,主机名,存储都是随机,不稳定的,并且Pod的创建与销毁也是无序的.这个设计决定了无状态服务并 不适合数据库领域的应用.

而Stateful管理有状态的应用,它的Pod有如下特征:

唯一性: 每个Pod会被分配一个唯一序号.
顺序性: Pod启动,更新,销毁是按顺序进行.
稳定的网络标识: Pod主机名,DNS地址不会随着Pod被重新调度而发生变化.
稳定的持久化存储: Pod被重新调度后,仍然能挂载原有的PV,从而保证了数据的完整性和一致性.



3.1. 如何理解稳定网络标识?

创建名为test-redis-pod的Stateful模型,根据你配置的Replica=3的设置,K8S会创建三个Pod,依次命名为: test-redis-pod-0; test-redis-pod-1; test-redis-pod-2

K8S为有状态的服务Pod分配稳定的网络标识,具体实现基于test-redis-pod-0名称,借助Headless DNS进行如下解析,获取后端其中一个Pod的地址.

$(pod name).$(service name).$(namespace).svc.cluster.local
下面是一个通过Pod名称访问Redis集群的Master节点地址的方法.

session.save_path = "tcp://test-redis-pod-0.test-redis-service.default.svc.cluster.local:6379"
现在回答如下二个问题:

在Redis Pod内部,主从节点之间数据同步的需求,Slave节点对应的配置文件中需要一个稳定的Master地址.下边脚本就是稳定访问test-redis-pod-0 名称来间接获得Redis Master节点IP地址,然后写入到Redis Slave的配置文件中,这样后续Slave节点与Master节点才能完成增量数据的同步.
if [ "${server_host}" != "test-redis-pod-0" ];then
        #echo "server-count: ${server_counts}" >> /data/test.log

        while [ -z "${master_address}" ];do
            echo "master_address is not available, ${master_address} waiting for redis master..." >> /data/test.log
            master_address=$(replication_master_address)
            sleep 1s
        done
    fi

    echo "master_address: $(master_address)" >> /data/test.log
    if [ ! -z "$master_address" ]; then
        printf "\nslaveof %s 6379\n" "$master_address" >> $conf
    fi
在Redis Pod外部, 可以通过$(pod name).$(service name).$(namespace).svc.cluster.local方式来访问具体的Pod服务.
3.2. 如何理解稳定持久化存储?





每个Redis Pod对应一个自己的PVC/PV.当Pod发生调度时,需要在别的节点启动时,根据Pod背后关联的存储信息保证其名称的稳定性.

Pod还是会attach挂载到原来的PV/PVC中,从而确定每个Pod有自己专用的存储卷.

4. 总结
本文主要介绍了无状态和有状态服务在K8S中的典型应用场景.

通过对Deployment部署无状态服务所遇到问题的分析,引出了Stateful新的部署组件.它是通过支持Pod一些特性(e.g. 名称唯一性,稳定的网络标识, 稳定的持久化存储等)来实现在K8S中部署运维有状态服务.

牢记: Stateful有状态服务,每个Pod有独立的PVC/PV存储组件


