---
title: serialization-thrift
date: 2018-09-06 09:25:56
tags:
categories:
---
# 官网
https://thrift.apache.org/docs/idl


# 简介

Thrift最初由Facebook研发，主要用于各个服务之间的RPC通信，支持跨语言，常用的语言比如C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, JavaScript, Node.js, Smalltalk, and OCaml都支持。Thrift是一个典型的CS（客户端/服务端）结构，客户端和服务端可以使用不同的语言开发。既然客户端和服务端能使用不同的语言开发，那么一定就要有一种中间语言来关联客户端和服务端的语言，没错，这种语言就是IDL（Interface Description Language）。

# idc
## 基本类型
thrift不支持无符号类型，因为很多编程语言不存在无符号类型，比如java

byte: 有符号字节
i16: 16位有符号整数
i32: 32位有符号整数
i64: 64位有符号整数
double: 64位浮点数
string: 字符串
## 容器类型
集合中的元素可以是除了service之外的任何类型，包括exception。

list<T>: 一系列由T类型的数据组成的有序列表，元素可以重复
set<T>: 一系列由T类型的数据组成的无序集合，元素不可重复
map<K, V>: 一个字典结构，key为K类型，value为V类型，相当于Java中的HMap<K,V>
## 结构体(struct)
就像C语言一样，thrift也支持struct类型，目的就是将一些数据聚合在一起，方便传输管理。struct的定 义形式如下：
```
struct People {
     1: string name;
     2: i32 age;
     3: string sex;
}
```

## 枚举(enum)
枚举的定义形式和Java的Enum定义差不多，例如：
```
enum Sex {
    MALE,
    FEMALE
}
```
## 异常(exception)
thrift支持自定义exception，规则和struct一样，如下：

```
exception RequestException {
    1: i32 code;
    2: string reason;
}
```
## 服务(service)
thrift定义服务相当于Java中创建Interface一样，创建的service经过代码生成命令之后就会生成客户端和服务端的框架代码。定义形式如下：
```
service HelloWordService {
     // service中定义的函数，相当于Java interface中定义的函数
     string doAction(1: string name, 2: i32 age);
 }
 ```
## 类型定义
thrift支持类似C++一样的typedef定义，比如：
```
typedef i32 Integer
typedef i64 Long
```
注意，末尾没有逗号或者分号

## 常量(const)
thrift也支持常量定义，使用const关键字，例如：
```
const i32 MAX_RETRIES_TIME = 10
const string MY_WEBSITE = "http://qifuguang.me";
```

## 注释
thrift注释方式支持shell风格的注释，支持C/C++风格的注释，即#和//开头的语句都单当做注释，/**/包裹的语句也是注释。

## 可选与必选
thrift提供两个关键字required，optional，分别用于表示对应的字段时必填的还是可选的。例如：

```
struct People {
    1: required string name;
    2: optional i32 age;
}
```
表示name是必填的，age是可选的。

# 生成代码

知道了怎么定义thirtf文件之后，我们需要用定义好的thrift文件生成我们需要的目标语言的源码，本文以生成java源码为例。假设现在定义了如下一个thrift文件：
```
namespace java com.winwill.thrift

enum RequestType {
   SAY_HELLO,   //问好
   QUERY_TIME,  //询问时间
}

struct Request {
   1: required RequestType type;  // 请求的类型，必选
   2: required string name;       // 发起请求的人的名字，必选
   3: optional i32 age;           // 发起请求的人的年龄，可选
}

exception RequestException {
   1: required i32 code;
   2: optional string reason;
}

// 服务名
service HelloWordService {
   string doAction(1: Request request) throws (1:RequestException qe); // 可能抛出异常。
}
```
在终端运行如下命令(前提是已经安装thrift)：
```
thrift --gen java Test.thrift
```
则在当前目录会生成一个gen-java目录，该目录下会按照namespace定义的路径名一次一层层生成文件夹
hrift文件中定义的enum，struct，exception，service都相应地生成了一个Java类，这就是能支持Java语言的基本的框架代码。

# 服务端实现
上面代码生成这一步已经将接口代码生成了，现在需要做的是实现HelloWordService的具体逻辑，实现的方式就是创建一个Java类，implements com.winwill.thrift.HelloWordService



# 参考
https://www.jianshu.com/p/0f4113d6ec4b