---
title: redis-persist
date: 2018-03-27 13:05:30
tags:
    - redis
categories:
    - redis
---
# PERSIST mykey
```
起始版本：2.2.0

时间复杂度：O(1)
```
移除给定key的生存时间，将这个 key 从『易失的』(带生存时间 key )转换成『持久的』(一个不带生存时间、永不过期的 key )。

##返回值

integer-reply, 只有以下两种值:

当生存时间移除成功时，返回 1 .
如果 key 不存在或 key 没有设置生存时间，返回 0 .
##例子

```
redis> SET mykey "Hello"
OK
redis> EXPIRE mykey 10
(integer) 1
redis> TTL mykey
(integer) 10
redis> PERSIST mykey
(integer) 1
redis> TTL mykey
(integer) -1
redis> 
```