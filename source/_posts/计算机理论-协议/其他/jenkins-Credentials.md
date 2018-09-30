---
title: jenkins-Credentials
abbrlink: 2068823166
date: 2018-04-20 12:58:56
tags:
categories:
---
# 背景
之前一直是运维部署的Jenkins,今天自己部署也遇到了些问题

# Credentials
  Jenkins的“Credentials”直译为“证书”，“文凭”在这里，尽管说Jenkins界面中有相当一部分的文字经过了官方汉化，但是对于这个“Credentials”却仍然没有被汉化，可见对于其汉化之后对应的中文翻译，官方也是一脸懵逼的状态，在这里，我们可以把它直译为“证书”，如果通过意译，那它的理解就是“钥匙”，这个翻译对于我们而言是最为容易理解的。

  如果要是把它理解为“钥匙”，那就不需要我进行过多的解释了，我们以SVN（版本控制器）为例来进行说明，当我们在访问SVN时，其实SVN是需要我们提供相应的账号与密码进行登录的，假如说我们把要访问的URL地址理解为锁，那么所提供的账号与密码就对应于开这把锁的钥匙，所以说“Credentials”中所记录的就是各种各样的这种钥匙，而这里的钥匙对应的锁是有多种可能的，可能是SVN，也可能是Git等，而“Credentials”就是对于这些钥匙所进行的统一管理。
  
  
![upload successful](/images/pasted-157.png)


# Credentials ID
Jenkins自动帮我们生成了一条“ID”信息,这个id就是用来在pipeline中对git项目进行登陆认证.

例如:
```
   stage 'Build'
   dir('gim') {
     git branch: BRANCH_NAME,credentialsId: 'fe65e7e9-2c93-48a9-81f8-db19ec174adc', url: 'http://xxx.git'
     sh "mvn  clean package -Dmaven.test.skip=true -U"
   }
```