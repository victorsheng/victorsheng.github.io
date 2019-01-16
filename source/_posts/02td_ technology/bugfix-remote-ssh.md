---
title: bugfix-remote-ssh
abbrlink: 3500998150
date: 2018-04-20 17:00:37
tags:
categories:
---
# 问题
通过ssh root@地址 "命令" 无法加载.bash_profile里面的配置

## 对比1
远程的ssh命令
```
[ip1:vic@ip1:/home/vic]$ ssh hadoop@ip2 "java -version"
java version "1.7.0_101"
OpenJDK Runtime Environment (rhel-2.6.6.1.el7_2-x86_64 u101-b00)
OpenJDK 64-Bit Server VM (build 24.95-b01, mixed mode)
ssh登陆后的
[ip2:hadoop@ip2:/home/hadoop]$ java -version
java version "1.8.0_66"
Java(TM) SE Runtime Environment (build 1.8.0_66-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.66-b17, mixed mode)
```

## 对比2
```
[ip1:vic@ip1:/home/vic]$ ssh hadoop@ip2 'echo $PATH'
/usr/local/bin:/usr/bin

[ip2:hadoop@ip2:/home/hadoop]$ echo $PATH
/usr/java/jdk1.8.0_66/bin:/usr/java/default/bin:/usr/java/default/jre/bin:/usr/local/mysql3306/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin

```

# 解决方案
- 设置BASH_ENV(没有找到设置的地方,export只会在本次登陆生效)
- 在远端要执脚本中加载配置文件(最终采取的方案)
```
#!/bin/bash
. ~/.bash_profile
```

# 参考文章
http://feihu.me/blog/2014/env-problem-when-ssh-executing-command-on-remote

# bash的四种模式
在man page的INVOCATION一节讲述了bash的四种模式，bash会依据这四种模式而选择加载不同的配置文件，而且加载的顺序也有所不同。本文ssh问题的答案就存在于这几种模式当中，所以在我们揭开谜底之前先来分析这些模式。

## interactive + login shell

第一种模式是交互式的登陆shell，这里面有两个概念需要解释：interactive和login：

login故名思义，即登陆，login shell是指用户以非图形化界面或者以ssh登陆到机器上时获得的第一个shell，简单些说就是需要输入用户名和密码的shell。因此通常不管以何种方式登陆机器后用户获得的第一个shell就是login shell。

interactive意为交互式，这也很好理解，interactive shell会有一个输入提示符，并且它的标准输入、输出和错误输出都会显示在控制台上。所以一般来说只要是需要用户交互的，即一个命令一个命令的输入的shell都是interactive shell。而如果无需用户交互，它便是non-interactive shell。通常来说如bash script.sh此类执行脚本的命令就会启动一个non-interactive shell，它不需要与用户进行交互，执行完后它便会退出创建的shell。

那么此模式最简单的两个例子为：

用户直接登陆到机器获得的第一个shell
用户使用ssh user@remote获得的shell

### 加载配置文件
这种模式下，shell首先加载/etc/profile，然后再尝试依次去加载下列三个配置文件之一，一旦找到其中一个便不再接着寻找：

```
~/.bash_profile
~/.bash_login
~/.profile
```


## non-interactive + login shell
第二种模式的shell为non-interactive login shell，即非交互式的登陆shell，这种是不太常见的情况。一种创建此shell的方法为：bash -l script.sh，前面提到过-l参数是将shell作为一个login shell启动，而执行脚本又使它为non-interactive shell。

对于这种类型的shell，配置文件的加载与第一种完全一样，在此不再赘述。

## interactive + non-login shell

bash，此时会打开一个交互式的shell，而因为不再需要登陆，因此不是login shell。

### 加载配置文件
对于此种情况，启动shell时会去查找并加载/etc/bash.bashrc和~/.bashrc文件。

### bashrc VS profile

从刚引入的两个配置文件的存放路径可以很容易的判断，第一个文件是全局性的，第二个文件属于当前用户。在前面的模式当中，已经出现了几种配置文件，多数是以profile命名的，那么为什么这里又增加两个文件呢？这样不会增加复杂度么？我们来看看此处的文件和前面模式中的文件的区别。

首先看第一种模式中的profile类型文件，它是某个用户唯一的用来设置全局环境变量的地方, 因为用户可以有多个shell比如bash, sh, zsh等, 但像环境变量这种其实只需要在统一的一个地方初始化就可以, 而这个地方就是profile，所以启动一个login shell会加载此文件，后面由此shell中启动的新shell进程如bash，sh，zsh等都可以由login shell中继承环境变量等配置。

接下来看bashrc，其后缀rc的意思为Run Commands，由名字可以推断出，此处存放bash需要运行的命令，但注意，这些命令一般只用于交互式的shell，通常在这里会设置交互所需要的所有信息，比如bash的补全、alias、颜色、提示符等等。

所以可以看出，引入多种配置文件完全是为了更好的管理配置，每个文件各司其职，只做好自己的事情。

## non-interactive + non-login shell
最后一种模式为非交互非登陆的shell，创建这种shell典型有两种方式：

bash script.sh
ssh user@remote command
这两种都是创建一个shell，执行完脚本之后便退出，不再需要与用户交互。

### 加载配置文件
对于这种模式而言，它会去寻找环境变量BASH_ENV，将变量的值作为文件名进行查找，如果找到便加载它。


# 配置文件的种类

- /etc/profile
- ~/.bash_profile
- ~/.bash_login
- ~/.profile
- /etc/bash.bashrc
- ~/.bashrc
- $BASH_ENV
- $ENV

# 小结

- ~/.bash_profile：应该尽可能的简单，通常会在最后加载.profile和.bashrc(注意顺序)
- ~/.bash_login：在前面讨论过，别用它
- ~/.profile：此文件用于login shell，所有你想在整个用户会话期间都有效的内容都应该放置于此，比如启动进程，环境变量等
- ~/.bashrc：只放置与bash有关的命令，所有与交互有关的命令都应该出现在此，比如bash的补全、alias、颜色、提示符等等。特别注意：别在这里输出任何内容（我们前面只是为了演示，别学我哈）
