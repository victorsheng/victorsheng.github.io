---
title: opitimize_git_clone
date: 2017-12-20 10:09:27
tags:
  - 代理
categories:
    - git
---
# 找到自己代理的端口
<img src="http://pic.victor123.cn/17-12-20/69754851.jpg">

# 命令:
git config --global http.https://github.com.proxy https://127.0.0.1:1087
git config --global https.https://github.com.proxy https://127.0.0.1:1087


# 查看git设置
git config --list

#BTW
此种加速对http和https协议有效 对ssh协议无效,如:
git clone git@github.com:xxxxxx/xxxxxx.git
