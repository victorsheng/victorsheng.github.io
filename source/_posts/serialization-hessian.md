---
title: serialization-hessian
date: 2018-09-05 18:48:20
tags:
categories:
---
# 官方英文文档
http://hessian.caucho.com/doc/hessian-serialization.html

# 介绍
Hessian是一种动态类型的、采用二进制序列化的，并且被设计为面向对象传输的Web服务协议。

# 设计目标
Hessian是动态类型的，紧凑的，可跨语言移植的。



# 语法
```
           # starting production
top        ::= value

           # 8-bit binary data split into 64k chunks
binary     ::= x41 b1 b0 <binary-data> binary # non-final chunk
           ::= 'B' b1 b0 <binary-data>        # final chunk
           ::= [x20-x2f] <binary-data>        # binary data of
                                                 #  length 0-15
           ::= [x34-x37] <binary-data>        # binary data of
                                                 #  length 0-1023

           # boolean true/false
boolean    ::= 'T'
           ::= 'F'

           # definition for an object (compact map)
class-def  ::= 'C' string int string*

           # time in UTC encoded as 64-bit long milliseconds since
           #  epoch
date       ::= x4a b7 b6 b5 b4 b3 b2 b1 b0
           ::= x4b b3 b2 b1 b0       # minutes since epoch

           # 64-bit IEEE double
double     ::= 'D' b7 b6 b5 b4 b3 b2 b1 b0
           ::= x5b                   # 0.0
           ::= x5c                   # 1.0
           ::= x5d b0                # byte cast to double
                                     #  (-128.0 to 127.0)
           ::= x5e b1 b0             # short cast to double
           ::= x5f b3 b2 b1 b0       # 32-bit float cast to double

           # 32-bit signed integer
int        ::= 'I' b3 b2 b1 b0
           ::= [x80-xbf]             # -x10 to x3f
           ::= [xc0-xcf] b0          # -x800 to x7ff
           ::= [xd0-xd7] b1 b0       # -x40000 to x3ffff

           # list/vector
list       ::= x55 type value* 'Z'   # variable-length list
	   ::= 'V' type int value*   # fixed-length list
           ::= x57 value* 'Z'        # variable-length untyped list
           ::= x58 int value*        # fixed-length untyped list
	   ::= [x70-77] type value*  # fixed-length typed list
	   ::= [x78-7f] value*       # fixed-length untyped list

           # 64-bit signed long integer
long       ::= 'L' b7 b6 b5 b4 b3 b2 b1 b0
           ::= [xd8-xef]             # -x08 to x0f
           ::= [xf0-xff] b0          # -x800 to x7ff
           ::= [x38-x3f] b1 b0       # -x40000 to x3ffff
           ::= x59 b3 b2 b1 b0       # 32-bit integer cast to long

           # map/object
map        ::= 'M' type (value value)* 'Z'  # key, value map pairs
	   ::= 'H' (value value)* 'Z'       # untyped key, value

           # null value
null       ::= 'N'

           # Object instance
object     ::= 'O' int value*
	   ::= [x60-x6f] value*

           # value reference (e.g. circular trees and graphs)
ref        ::= x51 int            # reference to nth map/list/object

           # UTF-8 encoded character string split into 64k chunks
string     ::= x52 b1 b0 <utf8-data> string  # non-final chunk
           ::= 'S' b1 b0 <utf8-data>         # string of length
                                             #  0-65535
           ::= [x00-x1f] <utf8-data>         # string of length
                                             #  0-31
           ::= [x30-x34] <utf8-data>         # string of length
                                             #  0-1023

           # map/list types for OO languages
type       ::= string                        # type name
           ::= int                           # type reference

           # main production
value      ::= null
           ::= binary
           ::= boolean
           ::= class-def value
           ::= date
           ::= double
           ::= int
           ::= list
           ::= long
           ::= map
           ::= object
           ::= ref
           ::= string
```


# 序列化

Hessian的对象序列化有8个原始类型:

- raw binary data
- boolean
- 64-bit millisecond date
- 64-bit double
- 32-bit int
- 64-bit long
- null
- UTF8-encoded string
它有3个递归类型:

- list for lists and arrays
- map for maps and dictionaries
- object for objects
最后，它有一个特殊的构造:

- ref for shared and circular object references.
Hessian 2.0 有三个内置引用类型映射:

- An object/list reference map.
- An class definition reference map.
- A type (class name) reference map.


## 二进制
### 二进制数据语法

```
binary ::= b b1 b0 <binary-data> binary
       ::= B b1 b0 <binary-data>
       ::= [x20-x2f] <binary-data>
```

二进制数据使用块编码。八进制 0x42('B')编码最后一个块，0x62('b')表示除了最后一个块以外的块。每个块都有一个16位的长度值。长度值公式：

len = 256 * b1 + b0

### 二进制示例
```
x20               # 0长度二进制数据

x23 x01 x02 x03   # 3个八字节数据

B x10 x00 ....    # 4k 结尾数据块  //256*16=4096

b x04 x00 ....    # 1k 非结尾数据块 //256*4=1028

```

## boolean
```
boolean ::= T
        ::= F
```
### Boolean 示例

T   # true
F   # false

## date
```
date ::= x4a b7 b6 b5 b4 b3 b2 b1 b0
     ::= x4b b4 b3 b2 b1 b0
```
日期类型使用一个64位的long数值表示从1970-01-01 00:00:00以来经过的毫秒数。
第二种方式是使用一个32位的int数值表示从1970-01-01 00:00以来经过的分钟数。

### date示例

```
x4a x00 x00 x00 xd0 x4b x92 x84 xb8   # 09:51:31 1998年5月8日UTC

x4b x4b x92 x0b xa0                   # 09:51:00 1998年5月8日UTC
```

## duoble



## int

## List
```
list ::= x55 type value* 'Z'   # variable-length list
     ::= 'V' type int value*   # fixed-length list
     ::= x57 value* 'Z'        # variable-length untyped list
     ::= x58 int value*        # fixed-length untyped list
     ::= [x70-77] type value*  # fixed-length typed list
     ::= [x78-7f] value*       # fixed-length untyped list
```

### 例子
Serialization of a typed int array: int[] = {0, 1}

```
V                    # fixed length, typed list
  x04 [int           # encoding of int[] type
  x92                # length = 2
  x90                # integer 0
  x91                # integer 1
```

untyped variable-length list = {0, 1}

```
x57                  # variable-length, untyped
  x90                # integer 0
  x91                # integer 1
  Z
```
## long

## objecy
```
class-def  ::= 'C' string int string*

object     ::= 'O' int value*
           ::= [x60-x6f] value*
```

### 对象示例
```
class Car {
  String color;
  String model;
}

out.writeObject(new Car("red", "corvette"));
out.writeObject(new Car("green", "civic"));

---

C                        # object definition (#0)
  x0b example.Car        # type is example.Car
  x92                    # two fields
  x05 color              # color field name
  x05 model              # model field name

O                        # object def (long form)
  x90                    # object definition #0
  x03 red                # color field value
  x08 corvette           # model field value

x60                      # object def #0 (short form)
  x05 green              # color field value
  x05 civic              # model field value
```


# 总结(by https://www.cnblogs.com/zhoukedou/p/6843504.html):
对性能敏感，对开发体验要求不高的内部系统。
　　thrift/protobuf

对开发体验敏感，性能有要求的内外部系统。
　　hessian2

对序列化后的数据要求有良好的可读性
　　jackson/gson/xml

