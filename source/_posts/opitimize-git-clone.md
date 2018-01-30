---
title: opitimize_git_clone
date: 2017-12-20 10:09:27
tags:
categories:
    - git
---
# 找到自己代理的端口
<img src="http://pic.victor123.cn/17-12-20/69754851.jpg">

# 命令:
git config --global http.https://github.com.proxy https://127.0.0.1:1087
git config --global https.https://github.com.proxy https://127.0.0.1:1087

作者：汪小九
链接：https://www.zhihu.com/question/27159393/answer/141047266
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

# 查看git设置
git config --list

#BTW
此种加速对http和https协议有效 对ssh协议无效,如:
git clone git@github.com:xxxxxx/xxxxxx.git
