---
title: 管理多个ssl私钥
abbrlink: 1584289957
date: 2018-04-10 11:29:00
tags:
categories:
---



# 背景
由于本地的sshkey已经绑定了gitlab,github等多个账号，如果换的话很麻烦
但是公司要求需要有公司邮箱的一个ssl私钥，so有了多个ssl管理的需求


# 步骤
大体参照https://www.cnblogs.com/popfisher/p/5731232.html 这篇文章即可


## 指定名称生成私钥
```
[appusr@iZ2zed4vtb8cdfk7pw69dzZ ~]$ ssh-keygen -t rsa -C "邮箱" -f newkey
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in newkey.
Your public key has been saved in newkey.pub.
The key fingerprint is:
bb:7a:24:8c:a7:ae:c8:81:6c:7a:b5:1a:82:19:84:d6 邮箱
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|. .              |
|.o E             |
|o                |
|.    o  S        |
|+o  o + ..       |
|=+.. + o.        |
|+oo.o   ..       |
|ooo+. .o.        |
+-----------------+


-rw-------  1 appusr appusr     1675 4月  10 19:36 newkey
-rw-r--r--  1 appusr appusr      388 4月  10 19:36 newkey.pub

```


## 配置.ssh/config文件
```
# 配置github.com
Host github.com                 
    HostName github.com
    IdentityFile C:\\Users\\popfisher\\.ssh\\id_rsa_github
    PreferredAuthentications publickey
    User username1

# 配置git.oschina.net 
Host git.oschina.net 
    HostName git.oschina.net
    IdentityFile C:\\Users\\popfisher\\.ssh\\id_rsa_oschina
    PreferredAuthentications publickey
    User username2
```

## 完成
只有指定的域名通过指定的ssl进行访问
其他域名，还是走默认的路径
`~/.ssh/id_rsa`