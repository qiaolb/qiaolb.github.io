---
layout: blog
title: Kubernetes加入containerd运行时节点
date: 2020-12-22 10:00:00
tags:
    - k8s
    - containerd
---

`Kubernetes`到`1.20`开始不建议使用`docker`作为运行时，作为容器运行时，其实还有其他一些：

* containerd
* CRI-O

经过对比，我选择了`containerd`进行验证。这次我并没有升级整体`Kubernetes`集群，只是打算增加一个使用`containerd`的容器运行时的节点。

## 安装`containerd`

我使用`CentOS 7`，首先需要设置内核参数，这个和之前`Docker`一样：

```
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# 设置必需的 sysctl 参数，这些参数在重新启动后仍然存在。
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

然后开始安装，`containerd`使用`yum`源和`docker-ce`相同：

```
# 安装 containerd
## 设置仓库
### 安装所需包
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

```
### 新增 Docker 仓库
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

```
## 安装 containerd
sudo yum update -y && sudo yum install -y containerd.io
```

## 配置`containerd`

首先获得缺省配置：

```
# 配置 containerd
sudo mkdir -p /etc/containerd
sudo containerd config default > /etc/containerd/config.toml
```

因为国内无法访问部分`image`源，同事`docker hub`比较慢，修改使用国内镜像：

```
vi /etc/containerd/config.toml

# 修改一下内容：
...
sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.2"
...
    [plugins."io.containerd.grpc.v1.cri".registry]
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://docker.mirrors.ustc.edu.cn"]
...
```

```
# 重启 containerd
sudo systemctl restart containerd
```

## 加入k8s集群

首先安装`k8s`，包括`kubeadm`、`kubelet`、`kubectl`

```
yum install -y kubeadm kubelet kubectl
```

然后加入到集群：

```
kubeadm join master.k8s:8443 --token xxx     --discovery-token-ca-cert-hash sha256:xxxxxxx
```

现在已经使用`containerd`允许时的节点已经加入到集群，部署容器测试可以正常使用。

## containerd管理

使用`containerd`已经不能使用熟悉的`docker`命令进行容器的管理了，`containerd`提供了2个工具可以使用:

* ctr
* crictl
  
### ctr

`ctr`是一个简单的命令工具，使用并不复杂，但需要注意：
* `ctr`不会使用配置文件`/etc/containerd/config.toml`，也就是说配置的`mirror`并不能使用
* `images`也有命名空间，`k8s`会使用一个名为`k8s.io`的命名空间
* `ctr`的参数有顺序，如`ctr -n=k8s.io images list`正确，而`ctr images list -n=k8s.io`则不正确
  
希望未来`ctr`会更完善一些。

### crictl

`crictl`使用和`docker`命令类似，比较方便。使用前需要增加配置文件：`/etc/crictl.yaml`。内容如下：

```
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
```


