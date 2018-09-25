---
title: 06-kubeadm
date: 2018-09-17 22:23:22
tags:
categories:
---
Kubernetes 的部署一直以来都是挡在初学者前面的一只“拦路虎”。

直到 2017 年，在志愿者的推动下，社区才终于发起了一个独立的部署工具，名叫：kubeadm。
这个项目的目的，就是要让用户能够通过这样两条指令完成一个 Kubernetes 集群的部署：

```
# 创建一个 Master 节点
$ kubeadm init

# 将一个 Node 节点加入到当前集群中
$ kubeadm join <Master 节点的 IP 和端口 >
```

# kubeadm 的工作原理

**把 kubelet 直接运行在宿主机上，然后使用容器部署其他的 Kubernetes 组件。**

```
$ apt-get install kubeadm
```

# kubeadm init 的工作流程
## 一系列的检查工作
当你执行 kubeadm init 指令后，kubeadm 首先要做的，是一系列的检查工作，以确定这台机器可以用来部署 Kubernetes。这一步检查，我们称为“Preflight Checks”，它可以为你省掉很多后续的麻烦。
其实，Preflight Checks 包括了很多方面，比如：
* Linux 内核的版本必须是否是 3.10 以上？
* Linux Cgroups 模块是否可用？
* 机器的 hostname 是否标准？在 Kubernetes 项目里，机器的名字以及一切存储在 Etcd 中的 API 对象，都必须使用标准的 DNS 命名（RFC 1123）。
* 用户安装的 kubeadm 和 kubelet 的版本是否匹配？
* 机器上是不是已经安装了 Kubernetes 的二进制文件？
* Kubernetes 的工作端口 10250/10251/10252 端口是不是已经被占用？
* ip、mount 等 Linux 指令是否存在？
* Docker 是否已经安装？
* ……

## 生成 Kubernetes 对外提供服务所需的各种证书和对应的目录。

在通过了 Preflight Checks 之后，kubeadm 要为你做的，是生成 Kubernetes 对外提供服务所需的各种证书和对应的目录。
Kubernetes 对外提供服务时，除非专门开启“不安全模式”，否则都要通过 HTTPS 才能访问 kube-apiserver。这就需要为 Kubernetes 集群配置好证书文件。
kubeadm 为 Kubernetes 项目生成的证书文件都放在 Master 节点的 /etc/kubernetes/pki 目录下。在这个目录下，最主要的证书文件是 ca.crt 和对应的私钥 ca.key。
此外，用户使用 kubectl 获取容器日志等 streaming 操作时，需要通过 kube-apiserver 向 kubelet 发起请求，这个连接也必须是安全的。kubeadm 为这一步生成的是 apiserver-kubelet-client.crt 文件，对应的私钥是 apiserver-kubelet-client.key。

## 生成kube-apiserver 所需的配置文件

证书生成后，kubeadm 接下来会为其他组件生成访问 kube-apiserver 所需的配置文件。这些文件的路径
是：/etc/kubernetes/xxx.conf：
```
ls /etc/kubernetes/
admin.conf  controller-manager.conf  kubelet.conf  scheduler.conf
```

**这些文件里面记录的是，当前这个 Master 节点的服务器地址、监听端口、证书目录等信息。这样，对应的客户端（比如 scheduler，kubelet 等），可以直接加载相应的文件，使用里面的信息与 kube-apiserver 建立安全连接。**





## kubeadm 会为 Master 组件生成 Pod 配置文件
接下来，kubeadm 会为 Master 组件生成 Pod 配置文件。我已经在上一篇文章中和你介绍过 Kubernetes 有三个 Master 组件 kube-apiserver、kube-controller-manager、kube-scheduler，而它们都会被使用 Pod 的方式部署起来。
你可能会有些疑问：这时，Kubernetes 集群尚不存在，难道 kubeadm 会直接执行 docker run 来启动这些容器吗？
当然不是。

## 启动kube-apiserver、kube-controller-manager、kube-scheduler

在 Kubernetes 中，有一种特殊的容器启动方法叫做“Static Pod”。它允许你把要部署的 Pod 的 YAML 文件放在一个指定的目录里。这样，当这台机器上的 kubelet 启动时，它会自动检查这个目录，加载所有的 Pod YAML 文件，然后在这台机器上启动它们。
从这一点也可以看出，kubelet 在 Kubernetes 项目中的地位非常高，在设计上它就是一个完全独立的组件，而其他 Master 组件，则更像是辅助性的系统容器。

在 kubeadm 中，Master 组件的 YAML 文件会被生成在 /etc/kubernetes/manifests 路径下。比如，kube-apiserver.yaml：

```
apiVersion: v1
kind: Pod
metadata:
 annotations:
    scheduler.alpha.kubernetes.io/critical-pod: ""
 creationTimestamp: null
 labels:
 component: kube-apiserver
 tier: control-plane
 name: kube-apiserver
 namespace: kube-system
spec:
 containers:
 - command:
 - kube-apiserver
 - --authorization-mode=Node,RBAC
 - --runtime-config=api/all=true
 - --advertise-address=10.168.0.2
    ...
 - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
 - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
 image: k8s.gcr.io/kube-apiserver-amd64:v1.11.1
 imagePullPolicy: IfNotPresent
 livenessProbe:
      ...
 name: kube-apiserver
 resources:
 requests:
 cpu: 250m
 volumeMounts:
 - mountPath: /usr/share/ca-certificates
 name: usr-share-ca-certificates
 readOnly: true
      ...
 hostNetwork: true
 priorityClassName: system-cluster-critical
 volumes:
 - hostPath:
 path: /etc/ca-certificates
 type: DirectoryOrCreate
 name: etc-ca-certificates
  ...
```
在这里，你只需要关注这样几个信息：
1. 这个 Pod 里只定义了一个容器，它使用的镜像是：k8s.gcr.io/kube-apiserver-amd64:v1.11.1 。这个镜像是 Kubernetes 官方维护的一个组件镜像。
2. 这个容器的启动命令（commands）是 kube-apiserver --authorization-mode=Node,RBAC …，这样一句非常长的命令。其实，它就是容器里 kube-apiserver 这个二进制文件再加上指定的配置参数而已。
3. 如果你要修改一个已有集群的 kube-apiserver 的配置，需要修改这个 YAML 文件。
4. 这些组件的参数也可以在部署时指定，我很快就会讲解到。



## 启动etcd
在这一步完成后，kubeadm 还会再生成一个 Etcd 的 Pod YAML 文件，用来通过同样的 Static Pod 的方式启动 Etcd。


所以，最后 Master 组件的 Pod YAML 文件如下所示：
```
$ ls /etc/kubernetes/manifests/
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
```


而一旦这些 YAML 文件出现在被 kubelet 监视的 /etc/kubernetes/manifests 目录下，kubelet 就会自动创建这些 YAML 文件中定义的 Pod，即 Master 组件的容器。
Master 容器启动后，kubeadm 会通过检查 localhost:6443/healthz 这个 Master 组件的健康检查 URL，等待 Master 组件完全运行起来。
## 生成 bootstrap token
然后，kubeadm 就会为集群生成一个 bootstrap token。在后面，只要持有这个 token，任何一个安装了 kubelet 和 kubadm 的节点，都可以通过 kubeadm join 加入到这个集群当中。

这个 token 的值和使用方法会，会在 kubeadm init 结束后被打印出来。

## 将 ca.crt 等 Master 节点的重要信息 存入etcd
在 token 生成之后，kubeadm 会将 ca.crt 等 Master 节点的重要信息，通过 ConfigMap 的方式保存在 Etcd 当中，
供后续部署 Node 节点使用。这个 ConfigMap 的名字是 cluster-info。

## 安装默认插件
kubeadm init 的最后一步，就是安装默认插件。Kubernetes 默认 kube-proxy 和 DNS 这两个插件是必须安装的。它们分别用来提供整个集群的服务发现和 DNS 功能。其实，这两个插件也只是两个容器镜像而已，所以 kubeadm 只要用 Kubernetes 客户端创建两个 Pod 就可以了。

# kubeadm join 的工作流程

这个流程其实非常简单，kubeadm init 生成 bootstrap token 之后，你就可以在任意一台安装了 kubelet 和 kubeadm 的机器上执行 kubeadm join 了。
可是，为什么执行 kubeadm join 需要这样一个 token 呢？
因为，任何一台机器想要成为 Kubernetes 集群中的一个节点，就必须在集群的 kube-apiserver 上注册。可是，要想跟 apiserver 打交道，这台机器就必须要获取到相应的证书文件（CA 文件）。可是，为了能够一键安装，我们就不能让用户去 Master 节点上手动拷贝这些文件。
所以，kubeadm 至少需要发起一次“不安全模式”的访问到 kube-apiserver，从而拿到保存在 ConfigMap 中的 cluster-info（它保存了 APIServer 的授权信息）。而 bootstrap token，扮演的就是这个过程中的安全验证的角色。
只要有了 cluster-info 里的 kube-apiserver 的地址、端口、证书，kubelet 就可以以“安全模式”连接到 apiserver 上，这样一个新的节点就部署完成了。
接下来，你只要在其他节点上重复这个指令就可以了。


# 配置 kubeadm 的部署参数
强烈推荐你在使用 kubeadm init 部署 Master 节点时，使用下面这条指令：
```
$ kubeadm init --config kubeadm.yaml
```
这时，你就可以给 kubeadm 提供一个 YAML 文件（比如，kubeadm.yaml），它的内容如下所示（我仅列举了主要部分）：
```
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.0
api:
 advertiseAddress: 192.168.0.102
 bindPort: 6443
  ...
etcd:
 local:
 dataDir: /var/lib/etcd
 image: ""
imageRepository: k8s.gcr.io
kubeProxy:
 config:
 bindAddress: 0.0.0.0
    ...
kubeletConfiguration:
 baseConfig:
 address: 0.0.0.0
    ...
networking:
 dnsDomain: cluster.local
 podSubnet: ""
 serviceSubnet: 10.96.0.0/12
nodeRegistration:
 criSocket: /var/run/dockershim.sock
  ...
```

通过制定这样一个部署参数配置文件，你就可以很方便地在这个文件里填写各种自定义的部署参数了。比如，我现在要指定 kube-apiserver 的参数，那么我只要在这个文件里加上这样一段信息：
```
...
apiServerExtraArgs:
  advertise-address: 192.168.0.103
  anonymous-auth: false
  enable-admission-plugins: AlwaysPullImages,DefaultStorageClass
  audit-log-path: /home/johndoe/audit.log

```

然后，kubeadm 就会使用上面这些信息替换 /etc/kubernetes/manifests/kube-apiserver.yaml 里的 command 字段里的参数了。
而这个 YAML 文件提供的可配置项远不止这些。比如，你还可以修改 kubelet 和 kube-proxy 的配置，修改 Kubernetes 使用的基础镜像的 URL（默认的k8s.gcr.io/xxx镜像 URL 在国内访问是有困难的），指定自己的证书文件，指定特殊的容器运行时等等。这些配置项，就留给你在后续实践中探索了。


# 总结
在今天的这次分享中，我重点介绍了 kubeadm 这个部署工具的工作原理和使用方法。紧接着，我会在下一篇文章中，使用它一步步地部署一个完整的 Kubernetes 集群。
从今天的分享中，你可以看到，kubeadm 的设计非常简洁。并且，它在实现每一步部署功能时，都在最大程度地重用 Kubernetes 已有的功能，这也就使得我们在使用 kubeadm 部署 Kubernetes 项目时，非常有“原生”的感觉，一点都不会感到突兀。
而 kubeadm 的源代码，直接就在 kubernetes/cmd/kubeadm 目录下，是 Kubernetes 项目的一部分。其中，app/phases 文件夹下的代码，对应的就是我在这篇文章中详细介绍的每一个具体步骤。


