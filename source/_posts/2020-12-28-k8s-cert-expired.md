---
layout: blog
title: Kubernetes证书过期
date: 2020-12-28 08:00:00
tags:
    - k8s
    - cert
---

今天访问`Kubernetes`时得到如下错误：

```
> kubectl get node     
error: You must be logged in to the server (Unauthorized)
```

昨天还正常今天无法访问，怀疑是证书到期了，可以直接看看`master`上的证书文件，文件位于`/etc/kubernetes/pki`中，执行命令：

```
> for item in `find /etc/kubernetes/pki -maxdepth 2 -name "*.crt"`;do openssl x509 -in $item -text -noout| grep Not;echo ======================$item===============;done

            Not Before: Dec 27 07:18:44 2019 GMT
            Not After : Dec 24 07:18:44 2029 GMT
======================/etc/kubernetes/pki/ca.crt===============
            Not Before: Dec 27 07:18:44 2019 GMT
            Not After : Dec 22 23:43:16 2021 GMT
======================/etc/kubernetes/pki/apiserver.crt===============
            Not Before: Dec 27 07:18:44 2019 GMT
            Not After : Dec 22 23:43:17 2021 GMT
======================/etc/kubernetes/pki/apiserver-kubelet-client.crt===============
            Not Before: Dec 27 07:18:45 2019 GMT
            Not After : Dec 24 07:18:45 2029 GMT
======================/etc/kubernetes/pki/front-proxy-ca.crt===============
            Not Before: Dec 27 07:18:45 2019 GMT
            Not After : Dec 22 23:43:17 2021 GMT
======================/etc/kubernetes/pki/front-proxy-client.crt===============
            Not Before: Dec 27 07:18:45 2019 GMT
            Not After : Dec 24 07:18:45 2029 GMT
======================/etc/kubernetes/pki/etcd/ca.crt===============
            Not Before: Dec 27 07:18:45 2019 GMT
            Not After : Dec 22 23:42:44 2021 GMT
======================/etc/kubernetes/pki/etcd/server.crt===============
            Not Before: Dec 27 07:18:45 2019 GMT
            Not After : Dec 22 23:42:44 2021 GMT
======================/etc/kubernetes/pki/etcd/peer.crt===============
            Not Before: Dec 27 07:18:45 2019 GMT
            Not After : Dec 22 23:42:45 2021 GMT
======================/etc/kubernetes/pki/etcd/healthcheck-client.crt===============
            Not Before: Dec 27 07:18:45 2019 GMT
            Not After : Dec 22 23:43:17 2021 GMT
======================/etc/kubernetes/pki/apiserver-etcd-client.crt===============
```

查看后发现的确到期，那么我们`renew`证书即可：

```
> kubeadm alpha certs renew all
Command "all" is deprecated, please use the same command under "kubeadm certs"
[renew] Reading configuration from the cluster...
[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
certificate the apiserver uses to access etcd renewed
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
certificate for liveness probes to healthcheck etcd renewed
certificate for etcd nodes to communicate with each other renewed
certificate for serving etcd renewed
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed

Done renewing certificates. You must restart the kube-apiserver, kube-controller-manager, kube-scheduler and etcd, so that they can use the new certificates.
```

从提示来看这个命令已经不建议使用，未来会使用`kubeadm certs`，这次就先这样，下次可以试试这个命令。

执行完成后，将`/etc/kubernetes/admin.conf`复制到`~/.kube/config`，就可以正常使用了。我没有重启`kube-apiserver, kube-controller-manager, kube-scheduler and etcd`前已经可以连接了，安全起见，还是重启一下。

可以连接后，也可以通过`k8s`命令查看证书状态：

```
> kubeadm certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Dec 28, 2021 00:06 UTC   364d                                    no      
apiserver                  Dec 28, 2021 00:06 UTC   364d            ca                      no      
apiserver-etcd-client      Dec 28, 2021 00:06 UTC   364d            etcd-ca                 no      
apiserver-kubelet-client   Dec 28, 2021 00:06 UTC   364d            ca                      no      
controller-manager.conf    Dec 28, 2021 00:06 UTC   364d                                    no      
etcd-healthcheck-client    Dec 28, 2021 00:06 UTC   364d            etcd-ca                 no      
etcd-peer                  Dec 28, 2021 00:06 UTC   364d            etcd-ca                 no      
etcd-server                Dec 28, 2021 00:06 UTC   364d            etcd-ca                 no      
front-proxy-client         Dec 28, 2021 00:06 UTC   364d            front-proxy-ca          no      
scheduler.conf             Dec 28, 2021 00:06 UTC   364d                                    no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Dec 24, 2029 07:18 UTC   8y              no      
etcd-ca                 Dec 24, 2029 07:18 UTC   8y              no      
front-proxy-ca          Dec 24, 2029 07:18 UTC   8y              no      
```