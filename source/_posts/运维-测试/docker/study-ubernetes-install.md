title: mac安装kubernetes
tags:
  - docker
  - kubernetes
categories: 
    - docker
date: 2017-12-21 07:17:00
---
https://github.com/jolestar/kubernetes-complete-course

---

# 容器的特性

- 让不标准的物品标准化（杂物，水，应用）
- 给上层工具提供标准化的操作方式，屏蔽细节
    - 包装，运输
    - add/remove/iterator（复用算法）
    - 应用管理和调度

---


# Docker (Moby)  如何实现应用标准化

| 问题        | 现状         |Docker 的方案  |
| -------------| ---------- |:-------------|
|安装包| war/jar，rpm/deb, src, bin|Image/Image Layer |
|运行环境|jvm，php, ruby, python| Image/Image Layer|
|进程启动方式  |web container, cmd, script |Image ENTRYPOINT/CMD|
|进程资源隔离限制| |Namespace/CGroup|
|进程文件路径冲突| |Chroot|
|端口冲突| 修改配置 |Network（IP per Container）|
|日志输出| 文件|stderr/stdout|
|安装包的仓库| nexus, rpm rep，ftp|Docker Registry|

-------

# Kubernetes 为何而生 - 云发展到一个新阶段

## IaaS 云解决了哪些问题
按需购买
接管硬件资源的运维
提供可编程接口来管理资源
提供 SDN，SDS 模拟硬件网络以及存储

## 特点
对应用无侵入
面向资源

> **用户从关注资源的运维转向关注应用的开发运维成本**


---
# Kubernetes 为何而生 - 容器的成熟奠定了基础

## 容器（Docker/Moby） 解决了哪些问题
应用安装包的标准化（Image）
应用进程的标准化（Container）


## 特点
单进程标准化



---
# 容器编排系统应运而生

我们需要一种 **面向应用（Application Oriented）** 的系统来降低服务端应用的开发部署和运维成本

> We wanted people to be able to **program** for the data center just like they program for their laptop --Ben Hindman

我们再引申一下，从开发延伸到部署运维

> We wanted people to be able to manager app for the data center just like they manager app on their laptop


---
# 官网地址
http://kubernetes.kansea.com/docs/whatisk8s/


---
# mac本地安装kubernetes(minikube)
http://kubernetes.kansea.com/docs/getting-started-guides/minikube/
- virtualBox安装
- minikube安装
- kubectl安装
- 需要ss的的代理,否则google的镜像无法下载
```
minikube delete

minikube start --docker-env HTTP_PROXY=http://192.168.99.1:1087 --docker-env HTTPS_PROXY=http://192.168.99.1:1087 --docker-env NO_PROXY=192.168.99.0/24

kubectl get pods --all-namespaces

等待ready

kubectl get nodes

```

<img src="http://pic.victor123.cn/17-12-21/46187591.jpg">

```
victordeMacBook-Pro:~ victor$ kubectl run nginx --image=nginx --port=80
deployment "nginx" created
victordeMacBook-Pro:~ victor$ kubectl expose deployment nginx --port=80 --type=NodePort --name=nginx-http
service "nginx-http" exposed
victordeMacBook-Pro:~ victor$ kubectl get pods
NAME                     READY     STATUS              RESTARTS   AGE
nginx-85dfb4bc54-72fbk   0/1       ContainerCreating   0          17s
victordeMacBook-Pro:~ victor$ kubectl get services
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP        20m
nginx-http   NodePort    10.99.7.195   <none>        80:32562/TCP   12s
victordeMacBook-Pro:~ victor$ minikube service nginx-http --url
Waiting, endpoint for service is not ready yet...
Waiting, endpoint for service is not ready yet...
Waiting, endpoint for service is not ready yet...
Waiting, endpoint for service is not ready yet...
^C
victordeMacBook-Pro:~ victor$ kubectl get nodes
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     <none>    22m       v1.8.0
victordeMacBook-Pro:~ victor$ kubectl get pods
NAME                     READY     STATUS    RESTARTS   AGE
nginx-85dfb4bc54-72fbk   1/1       Running   0          2m
victordeMacBook-Pro:~ victor$ minikube service nginx-http --url
http://192.168.99.100:32562
```

- tips:
- 本地的ss需要设置http代理,其中监听的地址从127.0.0.1改为0.0.0.0,这样虚拟机就可以访问到了,在这步卡了很久

# docker安装
https://yeasy.gitbooks.io/docker_practice/content/kubernetes/quickstart.html

![upload successful](/images/pasted-0.png)

这些服务大概分为三类：主节点服务、工作节点服务和其它服务。

## 主节点服务
apiserver 是整个系统的对外接口，提供 RESTful 方式供客户端和其它组件调用；

scheduler 负责对资源进行调度，分配某个 pod 到某个节点上；

controller-manager 负责管理控制器，包括 endpoint-controller（刷新服务和 pod 的关联信息）和 replication-controller（维护某个 pod 的复制为配置的数值）。

## 工作节点服务
kubelet 是工作节点执行操作的 agent，负责具体的容器生命周期管理，根据从数据库中获取的信息来管理容器，并上报 pod 运行状态等；

proxy 为 pod 上的服务提供访问的代理。

## 其它服务
Etcd 是所有状态的存储数据库；

gcr.io/google_containers/pause:0.8.0 是 Kubernetes 启动后自动 pull 下来的测试镜像。

----
https://yeasy.gitbooks.io/docker_practice/content/kubernetes/concepts.html

![upload successful](/images/pasted-1.png)