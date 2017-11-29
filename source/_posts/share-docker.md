---
title: docker学习分型
date: 2017-11-27 22:36:20
tags: 
    - docker
    - 技术分享
categories:
---
# 简介
Docker 是个划时代的开源项目，它彻底释放了计算虚拟化的威力，极大提高了应用的运行效率，降低了云计算资源供应的成本！使用 Docker，可以让应用的部署、测试和分发都变得前所未有的高效和轻松！

无论是应用开发者、运维人员、还是其他信息技术从业人员，都有必要认识和掌握 Docker，节约有限的时间。


# docker的由来
https://joshhu.gitbooks.io/docker_theory_install/content/

# 为什么要使用 Docker？
https://yeasy.gitbooks.io/docker_practice/content/introduction/why.html

# 基本概念
- 镜像（Image）
- 容器（Container）
- 仓库（Repository）




# 官方仓库
https://store.docker.com/
![ image.png](http://pic.victor123.cn/17-11-27/43305231.jpg)

# mac版本图形化客户端:kitematic
![ image.png](http://pic.victor123.cn/17-11-27/36671816.jpg)
Kitematic 完全自动化了 Docker 安装和设置过程，并提供了一个直观的图形用户接口（GUI）来在 Mac 上运行 Docker。Kitematic 集成了 Docker Machine 来在 Mac 上分发一个虚拟机并安装 Docker 引擎。

一旦安装成功，Kitematic GUI 便会启动，紧接着你可以立刻运行控制台中的镜像。你仅仅只需要在 Kitematic 搜索框键入镜像名就可以搜索任何在 Docker Hub 上存在的镜像。通过 GUI 你可以非常容易的创建、运行和管理你的容器，不需要使用命令行或者是在 Docker CLI 和 GUI之间来回切换。

Kitematic 也让Docker的一些高级特性使用更加方便，比如管理端口和配置 volumes。你可以方便的修改环境变量、查看日志，单机终端就可以进入容器，这些特性GUI都支持。
----


# 进阶
## 数据卷
## 网络管理
## 容器编排
    编排是一个新的词汇，经过阅读才明白编排指的是容器的集群化和调度。另一类含义指的是容器管理，负责管理容器化应用和组件任务。
Docker Swarm、Kubernetes、Marathon和Nomad


----
# 实践

# 快速安装
```
useradd appusr
passwd appusr
groupadd docker
usermod -aG docker appusr


yum install epel-release –y
yum install docker-io –y
systemctl start docker
systemctl enable docker
```


# 使用案例

## 本地的nginx
## 数据库
## jenkins

# 附录

##10张图带你深入理解Docker容器和镜像
http://dockone.io/article/783

## 菜鸟网
- http://www.runoob.com/docker/docker-hello-world.html
- Docker 安装 Nginx
- Docker 安装 PHP
- Docker 安装 MySQL
- Docker 安装 Tomcat
- Docker  安装 Python
- Docker 安装 Redis
- Docker 安装 MongoDB
- Docker 安装 Apache



