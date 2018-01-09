title: jenv安装(管理多个jdk版本)
date: 2018-01-06 08:03:10
tags:
categories:
---
1.安装jenv
``` shell
brew install jenv
```
2.oracle官网下载各版本jdk

3.安装本地
```
jenv add /Library/Java/JavaVirtualMachines/jdk-9.0.1.jdk/Contents/Home
jenv add /Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home
jenv add /Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home
```
4.列表jdk
```
jenv versions
```
* system (set by /Users/victor/.jenv/version)
  1.7
  1.7.0.80
  1.8
  1.8.0.151
  9.0
  9.0.1
  oracle64-1.7.0.80
  oracle64-1.8.0.151
  oracle64-9.0.1

5.选择jdk
```
jenv global 1.7
java -version
```
java version "1.7.0_80"
Java(TM) SE Runtime Environment (build 1.7.0_80-b15)
Java HotSpot(TM) 64-Bit Server VM (build 24.80-b11, mixed mode)