title: docker学习分型
tags:
  - 技术分享
  - docker
categories:
  - docker
date: 2017-11-27 22:36:00
---
# 简介
- Docker 是个划时代的开源项目，它彻底释放了计算虚拟化的威力，极大提高了应用的运行效率，降低了云计算资源供应的成本！使用 Docker，可以让应用的部署、测试和分发都变得前所未有的高效和轻松！

- 无论是应用开发者、运维人员、还是其他信息技术从业人员，都有必要认识和掌握 Docker，节约有限的时间。


# docker的由来
- Docker 是 PaaS 提供商 dotCloud 开源的一个基于 LXC 的高级容器引擎，由 Go 语言编写的，源代码托管在 github 而且居然只有 1W 行就完成了这些功能。

- Docker自2013年以来非常火热，无论是从 github 上的代码活跃度，还是Redhat在RHEL6.5中集成对Docker的支持, 就连 Google 的 Compute Engine 也支持 docker 在其之上运行。

- Docker设想是交付运行环境如同海运，OS如同一个货轮，每一个在OS基础上的软件都如同一个集装箱，用户可以通过标准化手段自由组装运行环境，同时集装箱的内容可以由用户自定义，也可以由专业人员制造。这样，交付一个软件，就是一系列标准化组件的集合的交付，如同乐高积木，用户只需要选择合适的积木组合，并且在最顶端署上自己的名字(最后个标准化组件是用户的app)。这也就是基于docker的PaaS产品的原型。

![ image.png](http://pic.victor123.cn/17-12-1/22516184.jpg)


# 为什么要使用 Docker？
- 更高效的利用系统资源
- 更快速的启动时间
- 一致的运行环境
- 持续交付和部署
- 更轻松的迁移
- 更轻松的维护和扩展



# 基本概念
- 镜像（Image）
    + Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。
- 容器（Container）
    + 镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
    + 容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 命名空间。因此容器可以拥有自己的 root 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。也因为这种隔离的特性，很多人初学 Docker 时常常会混淆容器和虚拟机。
- 仓库（Repository）
    + 一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。

# 基本命令
- docker run busybox echo "hello world"

<img src="http://pic.victor123.cn/17-12-1/3097858.jpg">
- docker ps


# 官方仓库
https://store.docker.com/
![ image.png](http://pic.victor123.cn/17-11-27/43305231.jpg)

# mac版本图形化客户端:kitematic
![ image.png](http://pic.victor123.cn/17-11-27/36671816.jpg)
Kitematic 完全自动化了 Docker 安装和设置过程，并提供了一个直观的图形用户接口（GUI）来在 Mac 上运行 Docker。Kitematic 集成了 Docker Machine 来在 Mac 上分发一个虚拟机并安装 Docker 引擎。

一旦安装成功，Kitematic GUI 便会启动，紧接着你可以立刻运行控制台中的镜像。你仅仅只需要在 Kitematic 搜索框键入镜像名就可以搜索任何在 Docker Hub 上存在的镜像。通过 GUI 你可以非常容易的创建、运行和管理你的容器，不需要使用命令行或者是在 Docker CLI 和 GUI之间来回切换。

Kitematic 也让Docker的一些高级特性使用更加方便，比如管理端口和配置 volumes。你可以方便的修改环境变量、查看日志，单机终端就可以进入容器，这些特性GUI都支持。
----
# dockerfile
Dockerfile是由一系列命令和参数构成的脚本，这些命令应用于基础镜像并最终创建一个新的镜像。
例如:
https://hub.docker.com/r/haocen/docker-hexo-with-hexo-admin/


# 进阶

## 数据卷

- 数据卷 是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性:
    + 数据卷 可以在容器之间共享和重用
    + 对 数据卷 的修改会立马生效
    + 对 数据卷 的更新，不会影响镜像
    + 数据卷 默认会一直存在，即使容器被删除

## 网络管理

Docker启动时，会自动在主机上创建一个docker0虚拟网桥，实际上是Linux的一个bridge,可以理解为一个软件交换机，它会而挂载到它的网口之间进行转发 当创建一个Docker容器的时候，同理会创建一对veth pair接口(当数据包发送到一个接口时，另外一个接口也可以收到相同的数据包)，这对接口一端在容器内，即eth0;另一端在本地并被挂载到docker0网桥，名称以veth开头。

## 容器安全
- 内核命名空间
- 控制组
控制组是 Linux 容器机制的另外一个关键组件，负责实现资源的审计和限制。
它提供了很多有用的特性；以及确保各个容器可以公平地分享主机的内存、CPU、磁盘 IO 等资源；当然，更重要的是，控制组确保了当容器内的资源使用产生压力时不会连累主机系统。
- 内核能力机制
```
大部分情况下，容器并不需要“真正的” root 权限，容器只需要少数的能力即可。为了加强安全，容器可以禁用一些没必要的权限。

    完全禁止任何 mount 操作；
    禁止直接访问本地主机的套接字；
    禁止访问一些文件系统的操作，比如创建新的设备、修改文件属性等；
    禁止模块加载。

这样，就算攻击者在容器中取得了 root 权限，也不能获得本地主机的较高权限，能进行的破坏也有限。

默认情况下，Docker采用白名单机制，禁用必需功能之外的其它权限。 当然，用户也可以根据自身需求来为 Docker 容器启用额外的权限。
```

## 容器编排

    编排是一个新的词汇，指的是容器的集群化和调度。另一类含义指的是容器管理，负责管理容器化应用和组件任务。
    <img src="http://pic.victor123.cn/17-11-29/58239834.jpg">

Docker Swarm、Kubernetes、Marathon和Nomad


-------

# 快速安装docker
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
```

docker run -p 80:80 --name mynginx -v $PWD/www:/www -v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf -v $PWD/logs:/wwwlogs -v $PWD/logs:/etc/nginx/logs -d nginx

nginx配置文件:
server {

    listen       80 default_server;
    
    location /api/iac {
        add_header 'Access-Control-Allow-Origin' "*";
        add_header 'Access-Control-Allow-Credentials' true;
         proxy_pass http://192.168.0.222:9060;
     }
    location /api/bi {
        add_header 'Access-Control-Allow-Origin' "*";
        add_header 'Access-Control-Allow-Credentials' true;
         proxy_pass http://192.168.0.106:9991;
     }
    location = / {
       proxy_connect_timeout 1800;
       proxy_read_timeout 1800;
       proxy_send_timeout 1800;
       proxy_pass http://192.168.0.106:8888/page/index.html;
    }
    location / {
    proxy_pass http://192.168.0.106:8099;
    }
    location /page {
       proxy_pass http://192.168.0.106:8888;
    }
    location /static {
       proxy_pass http://192.168.0.106:8888;
    }
    location /main {
       proxy_pass http://192.168.0.106:8888/page/index.html;
    }
    location /login{
       proxy_pass http://192.168.0.106:8888/page/login/index.html;
    }
}

```

## 用docker安装mysql数据库
准备工作
```
yum install git
mkdir /docker
chown -R appusr /docker

sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://cz3my8je.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

```
启动
```
要复制conf到制定目录下

docker run -p 3306:3306 --name mymysql -v /docker/mysql/conf/my.cnf:/etc/mysql/my.cnf -v /docker/mysql/logs:/logs -v /docker/mysql/data:/mysql_data -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6

docker ps -a

docker run -it --link mymysql:mysql --rm mysql sh -c 'exec mysql -h 172.18.0.1 -P 3306 -u root -p123456'

CREATE USER 'pig'@'%' IDENTIFIED BY '123456';
GRANT ALL ON *.* TO 'pig'@'%';
```


-------

# 附录

## 10张图带你深入理解Docker容器和镜像
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

## 学习资料
- https://joshhu.gitbooks.io/docker_theory_install/content/
- https://docs.docker.com/
- https://yeasy.gitbooks.io/docker_practice/content/introduction/why.html



