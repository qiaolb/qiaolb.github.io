---
layout: blog
title: Kubernetes证书过期(二)
date: 2022-01-15 08:00:00
tags:
    - k8s
    - cert
---

今天在部署`Kubernetes`的`Deployment`时，删除`Deployment`，发现不会自动删除`Replica Set`和`Pod`，开始怀疑是某个工作`node`故障，重启或者迁移到其他`node`问题一样。后来重启了`master`节点，发现有2个`master`启动无法注册`node`。


我的环境中有3个`master`节点，检查容器，发现`master01`启动正常，`master02`和`master03`的`etcd`容器都无法启动，查看日志发现提示如下：

```
Jan 14 22:29:29 master02.k8s kubelet[1998]: E0114 22:29:29.187573    1998 kubelet.go:2291] "Error getting node" err="node \"master02.k8s\" not found"

```

在`master01`上查看`etcd`的日志发现：

```
2022-01-14 15:57:13.963722 I | embed: rejected connection from "192.168.203.4:45008" (error "tls: failed to verify client's certificate: x509: certificate has expired or is not yet valid", ServerName "")
```

明确是证书过期导致的，根据[上次过期的经](/k8s-cert-expired.html)验检查如下：

```
# kubeadm certs renew
missing subcommand; "renew" is not meant to be run on its own
To see the stack trace of this error execute with --v=5 or higher
[root@master02 kubernetes]# kubeadm certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[check-expiration] Error reading configuration from the Cluster. Falling back to default configuration

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Sep 10, 2022 00:10 UTC   238d                                    no      
apiserver                  Sep 10, 2022 00:20 UTC   238d            ca                      no      
apiserver-etcd-client      Sep 10, 2022 00:20 UTC   238d            etcd-ca                 no      
apiserver-kubelet-client   Sep 10, 2022 00:20 UTC   238d            ca                      no      
controller-manager.conf    Sep 10, 2022 00:09 UTC   238d                                    no      
etcd-healthcheck-client    Dec 22, 2021 23:53 UTC   <invalid>       etcd-ca                 no      
etcd-peer                  Dec 22, 2021 23:53 UTC   <invalid>       etcd-ca                 no      
etcd-server                Dec 22, 2021 23:53 UTC   <invalid>       etcd-ca                 no      
front-proxy-client         Sep 10, 2022 00:20 UTC   238d            front-proxy-ca          no      
scheduler.conf             Sep 10, 2022 00:09 UTC   238d                                    no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Dec 24, 2029 07:18 UTC   7y              no      
etcd-ca                 Dec 24, 2029 07:18 UTC   7y              no      
front-proxy-ca          Dec 24, 2029 07:18 UTC   7y              no      

```

可以看出`etcd`的证书的确过期，`renew`证书：

```
# kubeadm certs renew healthcheck-client
# kubeadm certs renew etcd-peer
# kubeadm certs renew etcd-server
```

重新检查确定新的证书已经生效，重启容器和`kubelet`，检查发现问题件已经解决。

产生这个问题的原因是，`master01`已经自动更新了证书，保证容器环境正常使用，但其他`master`节点的证书没有自动更新，需要手工处理一下，估计为了版本会解决这个问题。