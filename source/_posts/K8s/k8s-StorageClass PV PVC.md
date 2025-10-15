---
title: k8s-StorageClass PV PVC 概念和使用
categories:
  - k8s
date: 2024-04-12 10:25:53
tags: k8s
cover: /img/k8s.jpg
---
负责把PVC绑定到PV的是一个持久化存储卷控制循环，这个控制器也是kube-manager-controller的一部分运行在master上。而真正把目录挂载到容器上的操作是在POD所在主机上发生的，所以通过kubelet来完成。而且创建PV以及PVC的绑定是在POD被调度到某一节点之后进行的，完成这些操作，POD就可以运行了。下面梳理一下挂载一个PV的过程：

用户提交一个包含PVC的POD

调度器把根据各种调度算法把该POD分配到某个节点，比如node01

Node01上的kubelet等待Volume Manager准备存储设备

PV控制器调用存储插件创建PV并与PVC进行绑定

Attach/Detach Controller或Volume Manager通过存储插件实现设备的attach。（这一步是针对块设备存储）

Volume Manager等待存储设备变为可用后，挂载该设备到/var/lib/kubelet/pods/<Pod 的 ID>/volumes/kubernetes.io~<Volume 类型 >/<Volume 名字 >目录上

Kubelet被告知卷已经准备好，开始启动POD，通过映射方式挂载到容器中

总结：本地卷也就是LPV不支持动态供给的方式，延迟绑定，就是为了综合考虑所有因素再进行POD调度。其根本原因是动态供给是先调度POD到节点，然后动态创建PV以及绑定PVC最后运行POD；而LPV是先创建与某一节点关联的PV，然后在调度的时候综合考虑各种因素而且要包括PV在哪个节点，然后再进行调度，到达该节点后在进行PVC的绑定。也就说动态供给不考虑节点，LPV必须考虑节点。所以这两种机制有冲突导致无法在动态供给策略下使用LPV。换句话说动态供给是PV跟着POD走，而LPV是POD跟着PV走。