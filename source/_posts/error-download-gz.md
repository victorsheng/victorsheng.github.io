---
title: mac_Chrome浏览器下载csv.gz后缀时遇到的问题
date: 2017-12-04 22:34:00
tags:
categories:
---
# 问题描述:
- 下载一个应该为csv.gz后缀名的问题,不知道为什么没有了后缀名,一开始以为是字符集问题
- 下载的是kaggle的data数据
https://www.kaggle.com/c/expedia-hotel-recommendations/data

<img src="http://pic.victor123.cn/17-12-4/19333252.jpg">

- 报错错误
<img src="http://pic.victor123.cn/17-12-4/81379727.jpg">
- 问了小伙伴,解压后的文件应该有几个g,不应该几百兆,因此推断是后缀名问题

# 解决

http://pic.victor123.cn/17-12-4/7396408.jpg

# 知识小结
```

.tar 
解包：tar xvf FileName.tar
打包：tar cvf FileName.tar DirName
（注：tar是打包，不是压缩！）
———————————————
.gz
解压1：gunzip FileName.gz
解压2：gzip -d FileName.gz
压缩：gzip FileName

.tar.gz 和 .tgz
解压：tar zxvf FileName.tar.gz
压缩：tar zcvf FileName.tar.gz DirName
———————————————

```

