---
title: serialization-protobuf
abbrlink: 313737339
date: 2018-09-05 22:33:36
tags:
categories:
---
# 介绍
Protocol buffers是Google的语言中立，平台中立，可扩展的机制，用于序列化结构化数据 - 思考XML，但更小，更快，更简单。 您可以定义数据的结构化时间，然后可以使用特殊生成的源代码轻松地在各种数据流中使用各种语言编写和读取结构化数据。

# 官网
https://developers.google.com/protocol-buffers/docs/overview


https://developers.google.com/protocol-buffers/docs/reference/proto2-spec

https://developers.google.com/protocol-buffers/docs/reference/proto3-spec



# demo

```
message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;
}


```

# v2 vs v3
```
syntax="proto3";

message Person {
    int64 id = 1;
    string name = 2;
}

------------------------------------------------

message Person {
    int64 id = 1;
    string name = 2;
}
```

# 语法
## message
message 用来定义一个数据结构
- 命名：常规的命名方式建议使用驼峰法，即：HelloWorld 样式
- 注释： message 中支持 // 这样的单行注释
- repeated 的使用：被 repeated 标识的字段可以理解为是一个数组,示例：

```
message Person {
    int64 id = 1;
    string name = 2;
    repeated string skills = 3;  // 这里表示 skills 可以接受多个 string 类型的值
}
```
枚举用来表示一定范围内具有相同属性的值，示例：
```
syntax = "proto3";
message Employee {
    int64 id = 1;
    string name = 2;
    enum Skills {
        GOLANG = 0;
        PYTHON = 1;
        JAVA = 2;
        RUST = 3;
        CPP = 4;
    }
    Skills skill = 3;
}

```
声明自己定义的 message 类型，示例：
```
syntax = "proto3";
message Employee {
    int64 id = 1;
    string name = 2;
    Skill skills = 3;  //这里声明的为自定义的 Skill 类型
}
message Skill {
    string name = 1;
}

```
message 定义时可以使用 map 类型，示例：
```
syntax = "proto3";
message Employee {
    int64 id = 1;
    string name = 2;
    map<string, Skill> skills = 3;
}
message Skill {
    string name = 1;
}

```

## package
每个 *.proto 文件可以指定 package 作为生成语言的 namespace，示例：
```
syntax = "proto3";
package exmple;
message Employee {
    int64 id = 1;
    string name = 2;
    map<string, Skill> skills = 3;
}
message Skill {
    string name = 1;
}

```

# 参考
https://colobu.com/2015/01/07/Protobuf-language-guide/

https://solicomo.com/network-dev/protobuf-proto3-vs-proto2.html