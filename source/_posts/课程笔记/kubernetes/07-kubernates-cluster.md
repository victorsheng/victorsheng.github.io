---
title: 07-kubernates-cluster
abbrlink: 881185341
date: 2018-09-19 07:10:55
tags:
categories:
---
# 整体步骤
1. 在所有节点上安装 Docker 和 kubeadm；
2. 部署 Kubernetes Master；
3. 部署容器网络插件；
4. 部署 Kubernetes Worker；
5. 部署 Dashboard 可视化插件；
6. 部署容器存储插件。

# 安装 kubeadm 和 Docker
root用户下
```
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
$ cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
$ apt-get update
$ apt-get install -y docker.io kubeadm
```

**在上述安装 kubeadm 的过程中，kubeadm 和 kubelet、kubectl、kubernetes-cni 这几个二进制文件都会被自动安装好。**


# 部署 Kubernetes 的 Master 节点
kubeadm 用的 YAML 文件（名叫：kubeadm.yaml）：

```
apiVersion: kubeadm.k8s.io/v1alpha1
kind: MasterConfiguration
controllerManagerExtraArgs:
  horizontal-pod-autoscaler-use-rest-clients: "true"
  horizontal-pod-autoscaler-sync-period: "10s"
  node-monitor-grace-period: "10s"
apiServerExtraArgs:
  runtime-config: "api/all=true"
kubernetesVersion: "stable-1.11"
```
## kubeadm init
然后，我们只需要执行一句指令：
```
$ kubeadm init --config kubeadm.yaml
```
就可以完成 Kubernetes Master 的部署了，这个过程只需要几分钟。部署完成后，kubeadm 会生成一行指令：

## kubeadm join
```
kubeadm join 10.168.0.2:6443 --token 00bwbx.uvnaa2ewjflwu1ry --discovery-token-ca-cert-hash sha256:00eb62a2a6020f94132e3fe1ab721349bbcd3e9b94da9654cfe15f2985ebd711
```
这个 kubeadm join 命令，就是用来给这个 Master 节点添加更多工作节点（Worker）的命令。我们在后面部署 Worker 节点的时候马上会用到它，所以找一个地方把这条命令记录下来。
此外，kubeadm 还会提示我们第一次使用 Kubernetes 集群所需要的配置命令：
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
而需要这些配置命令的原因是：Kubernetes 集群默认需要加密方式访问。所以，这几条命令，就是将刚刚部署生成的 Kubernetes 集群的安全配置文件，保存到当前用户的.kube 目录下，kubectl 默认会使用这个目录下的授权信息访问 Kubernetes 集群。


## 查看当前唯一一个节点的状态
现在，我们就可以使用 kubectl get 命令来查看当前唯一一个节点的状态了：
```
$ kubectl get nodes
NAME      STATUS     ROLES     AGE       VERSION
master    NotReady   master    1d        v1.11.1


$ kubectl describe node master

...
Conditions:
...

Ready   False ... KubeletNotReady  runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized


$ kubectl get pods -n kube-system
NAME               READY   STATUS   RESTARTS  AGE
coredns-78fcdf6894-j9s52     0/1    Pending  0     1h
coredns-78fcdf6894-jm4wf     0/1    Pending  0     1h
etcd-master           1/1    Running  0     2s
kube-apiserver-master      1/1    Running  0     1s
kube-controller-manager-master  0/1    Pending  0     1s
kube-proxy-xbd47         1/1    NodeLost  0     1h
kube-scheduler-master      1/1    Running  0     1s

```
可以看到，CoreDNS、kube-controller-manager 等依赖于网络的 Pod 都处于 Pending 状态，即调度失败。这当然是符合预期的：因为这个 Master 节点的网络尚未就绪。

# 部署网络插件
```
$ kubectl apply -f https://git.io/weave-kube-1.6
```
可以看到，所有的系统 Pod 都成功启动了，而刚刚部署的 Weave 网络插件则在 kube-system 下面新建了一个名叫 weave-net-cmk27 的 Pod，一般来说，这些 Pod 就是容器网络插件在每个节点上的控制组件。
Kubernetes 支持容器网络插件，使用的是一个名叫 CNI 的通用接口，它也是当前容器网络的事实标准，市面上的所有容器网络开源项目都可以通过 CNI 接入 Kubernetes，比如 Flannel、Calico、Canal、Romana 等等，它们的部署方式也都是类似的“一键部署”。关于这些开源项目的实现细节和差异，我会在后续的网络部分详细介绍。
至此，Kubernetes 的 Master 节点就部署完成了。如果你只需要一个单节点的 Kubernetes，现在你就可以使用了。不过，在默认情况下，Kubernetes 的 Master 节点是不能运行用户 Pod 的，所以还需要额外做一个小操作。在本篇的最后部分，我会介绍到它。

# 部署 Kubernetes 的 Worker 节点

Kubernetes 的 Worker 节点跟 Master 节点几乎是相同的，它们运行着的都是一个 kubelet 组件。唯一的区别在于，在 kubeadm init 的过程中，kubelet 启动后，Master 节点上还会自动运行 kube-apiserver、kube-scheduler、kube-controller-manger 这三个系统 Pod。
所以，相比之下，部署 Worker 节点反而是最简单的，只需要两步即可完成。
第一步，在所有 Worker 节点上执行“安装 kubeadm 和 Docker”一节的所有步骤。
第二步，执行部署 Master 节点时生成的 kubeadm join 指令：
```
$ kubeadm join 10.168.0.2:6443 --token 00bwbx.uvnaa2ewjflwu1ry --discovery-token-ca-cert-hash sha256:00eb62a2a6020f94132e3fe1ab721349bbcd3e9b94da9654cfe15f2985ebd711
```

# 通过 Taint/Toleration 调整 Master 执行 Pod 的策略
我在前面提到过，默认情况下 Master 节点是不允许运行用户 Pod 的。而 Kubernetes 做到这一点，依靠的是 Kubernetes 的 Taint/Toleration 机制。
它的原理非常简单：一旦某个节点被加上了一个 Taint，即被“打上了污点”，那么所有 Pod 就都不能在这个节点上运行，因为 Kubernetes 的 Pod 都有“洁癖”。
除非，有个别的 Pod 声明自己能“容忍”这个“污点”，即声明了 Toleration，它才可以在这个节点上运行。
其中，为节点打上“污点”（Taint）的命令是：
```
$ kubectl taint nodes node1 foo=bar:NoSchedule
```
这时，该 node1 节点上就会增加一个键值对格式的 Taint，即：foo=bar:NoSchedule。其中值里面的 NoSchedule，意味着这个 Taint 只会在调度新 Pod 时产生作用，而不会影响已经在 node1 上运行的 Pod，哪怕它们没有 Toleration。
那么 Pod 又如何声明 Toleration 呢？
我们只要在 Pod 的.yaml 文件中的 spec 部分，加入 tolerations 字段即可：
```
apiVersion: v1
kind: Pod
...
spec:
  tolerations:
  - key: "foo"
    operator: "Equal"
    value: "bar"
    effect: "NoSchedule"
```
这个 Toleration 的含义是，这个 Pod 能“容忍”所有键值对为 foo=bar 的 Taint（ operator: “Equal”，“等于”操作）。
现在回到我们已经搭建的集群上来。这时，如果你通过 kubectl describe 检查一下 Master 节点的 Taint 字段，就会有所发现了：
```
$ kubectl describe node master

Name:               master
Roles:              master
Taints:             node-role.kubernetes.io/master:NoSchedule
```
可以看到，Master 节点默认被加上了node-role.kubernetes.io/master:NoSchedule这样一个“污点”，其中“键”是node-role.kubernetes.io/master，而没有提供“值”。
此时，你就需要像下面这样用“Exists”操作符（operator: “Exists”，“存在”即可）来说明，该 Pod 能够容忍所有以 foo 为键的 Taint，才能让这个 Pod 运行在该 Master 节点上：
```
apiVersion: v1
kind: Pod
...
spec:
  tolerations:
  - key: "foo"
    operator: "Exists"
    effect: "NoSchedule"
```



## 如果你就是想要一个单节点的 Kubernetes，删除这个 Taint 才是正确的选择：
```
$ kubectl taint nodes --all node-role.kubernetes.io/master-
```

如上所示，我们在“node-role.kubernetes.io/master”这个键后面加上了一个短横线“-”，这个格式就意味着移除所有以“node-role.kubernetes.io/master”为键的 Taint。

**有了 kubeadm 这样的原生管理工具，Kubernetes 的部署已经被大大简化。更重要的是，像证书、授权、各个组件的配置等部署中最麻烦的操作，kubeadm 都已经帮你完成了。**

# 部署 Dashboard 可视化插件
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

部署完成之后，我们就可以查看 Dashboard 对应的 Pod 的状态了：
```
$ kubectl get pods -n kube-system
kubernetes-dashboard-6948bdb78-f67xk   1/1       Running   0          1m
```

需要注意的是，由于 Dashboard 是一个 Web Server，很多人经常会在自己的公有云上无意地暴露 Dashboard 的端口，从而造成安全隐患。所以，1.7 版本之后的 Dashboard 项目部署完成后，默认只能通过 Proxy 的方式在本地访问。具体的操作，你可以查看 Dashboard 项目的官方文档。
而如果你想从集群外访问这个 Dashboard 的话，就需要用到 Ingress，我会在后面的文章中专门介绍这部分内容。


# 部署容器存储插件Rook

我在前面介绍容器原理时已经提到过，很多时候我们需要用数据卷（Volume）把外面宿主机上的目录或者文件挂载进容器的 Mount Namespace 中，从而达到容器和宿主机共享这些目录或者文件的目的。容器里的应用，也就可以在这些数据卷中新建和写入文件。

可是，如果你在某一台机器上启动的一个容器，显然无法看到其他机器上的容器在它们的数据卷里写入的文件。这是容器最典型的特征之一：无状态。

**而容器的持久化存储，就是用来保存容器存储状态的重要手段：存储插件会在容器里挂载一个基于网络或者其他机制的远程数据卷，使得在容器里创建的文件，实际上是保存在远程存储服务器上，或者以分布式的方式保存在多个节点上，而与当前宿主机没有任何绑定关系。这样，无论你在其他哪个宿主机上启动新的容器，都可以请求挂载指定的持久化存储卷，从而访问到数据卷里保存的内容。这就是“持久化”的含义。**


由于 Kubernetes 本身的松耦合设计，绝大多数存储项目，比如 Ceph、GlusterFS、NFS 等，都可以为 Kubernetes 提供持久化存储能力。在这次的部署实战中，我会选择部署一个很重要的 Kubernetes 存储插件项目：Rook。

Rook 项目是一个基于 Ceph 的 Kubernetes 存储插件（它后期也在加入对更多存储实现的支持）。不过，不同于对 Ceph 的简单封装，Rook 在自己的实现中加入了水平扩展、迁移、灾难备份、监控等大量的企业级功能，使得这个项目变成了一个完整的、生产级别可用的容器存储插件。
得益于容器化技术，用两条指令，Rook 就可以把复杂的 Ceph 存储后端部署起来：
```
$ kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/operator.yaml

$ kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/cluster.yaml
```


在部署完成后，你就可以看到 Rook 项目会将自己的 Pod 放置在由它自己管理的两个 Namespace 当中：

```
$ kubectl get pods -n rook-ceph-system
NAME                                  READY     STATUS    RESTARTS   AGE
rook-ceph-agent-7cv62                 1/1       Running   0          15s
rook-ceph-operator-78d498c68c-7fj72   1/1       Running   0          44s
rook-discover-2ctcv                   1/1       Running   0          15s

$ kubectl get pods -n rook-ceph
NAME                   READY     STATUS    RESTARTS   AGE
rook-ceph-mon0-kxnzh   1/1       Running   0          13s
rook-ceph-mon1-7dn2t   1/1       Running   0          2s
```
这样，一个基于 Rook 持久化存储集群就以容器的方式运行起来了，而接下来在 Kubernetes 项目上创建的所有 Pod 就能够通过 Persistent Volume（PV）和 Persistent Volume Claim（PVC）的方式，在容器里挂载由 Ceph 提供的数据卷了。

**而 Rook 项目，则会负责这些数据卷的生命周期管理、灾难备份等运维工作。关于这些容器持久化存储的知识，我会在后续章节中专门讲解。**

# 总结
在本篇文章中，我们完全从 0 开始，在 Bare-metal 环境下使用 kubeadm 工具部署了一个完整的 Kubernetes 集群：这个集群有一个 Master 节点和多个 Worker 节点；使用 Weave 作为容器网络插件；使用 Rook 作为容器持久化存储插件；使用 Dashboard 插件提供了可视化的 Web 界面。
这个集群，也将会是我进行后续讲解所依赖的集群环境，并且在后面的讲解中，我还会给它安装更多的插件，添加更多的新能力。
另外，这个集群的部署过程并不像传说中那么繁琐，这主要得益于：
1. kubeadm 项目大大简化了部署 Kubernetes 的准备工作，尤其是配置文件、证书、二进制文件的准备和制作，以及集群版本管理等操作，都被 kubeadm 接管了。
2. Kubernetes 本身“一切皆容器”的设计思想，加上良好的可扩展机制，使得插件的部署非常简便。





