---
layout: blog
title: Kubernetes中删除Calico
date: 2019-09-03 10:00:00
tags:
    - k8s
    - network
    - calico
---

最近发现`Kubernetes`集群中，出现不稳定情况，怀疑是`Calico`导致，为了排除文件，打算将`Calico`更换为`Flannel`，本来以为很简单，结果还是遇到`Calico`无法删除干净的问题。所有将删除正确删除过程做一个记录。 顺便说一下，最终确定和`Calico`无关。

环境：
`Kubernetes： V1.15`
`Calico： V3.5`

删除步骤：

1. 删除`K8s`对象
  ```shell
  kubectl delete -f calico.yaml
  ```
2. 检查所有节点上的网络，看看是否存在`Tunl0`
  ```shell
  ip addr show
  ```
3. 如果有`Tunl0`，将其删除
  ```shell
  modprobe -r ipip
  ```
4. 移除`Calico`配置文件
  ```shell
  ls /etc/cni/net.d/
  ```
  看看是否存在`Calico`相关的文件和目录，如：`10-calico.conflist`，  `calico-kubeconfig`，  `calico-tls`，如果有将其移除。

这时候整个`Calico`移除成功。