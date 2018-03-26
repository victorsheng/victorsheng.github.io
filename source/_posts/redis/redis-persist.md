---
layout: "post"
title: "Redis持久化方式"
date: "2018-03-26 18:21"
---
为防止数据丢失，需要将 Redis 中的数据从内存中 dump 到磁盘，这就是持久化。Redis 提供两种持久化方式：RDB 和 AOF。Redis 允许两者结合，也允许两者同时关闭。

RDB 可以定时备份内存中的数据集。服务器启动的时候，可以从 RDB 文件中恢复数据集。

AOF\(append only file\) 可以记录服务器的所有写操作。在服务器重新启动的时候，会把所有的写操作重新执行一遍，从而实现数据备份。当写操作集过大（比原有的数据集还大），Redis 会重写写操作集。

为什么称为 append only file 呢？AOF 持久化是类似于生成一个关于 Redis 写操作的文件，写操作（增删）总是以追加的方式追加到文件中。



因此，在 RDB 持久化的时候可以将 RDB 保存到磁盘中，也可以保存到内存中，当然保存到内存中就不是持久化了。

## RDB 持久化的运作机制\(快照模式\) {#bc4ea917246862249c62979de56b1a87}

![](http://wiki.jikexueyuan.com/project/redis/images/redis16.png)

Redis 支持两种方式进行 RDB 持久化：当前进程执行和后台执行（BGSAVE）。RDB BGSAVE 策略是 fork 出一个子进程，把内存中的数据集整个 dump 到硬盘上。两个场景举例：

1. Redis 服务器初始化过程中，设定了定时事件，每隔一段时间就会触发持久化操作； 进入定时事件处理程序中，就会 fork 产生子进程执行持久化操作。
2. Redis 服务器预设了 save 指令，客户端可要求服务器进程中断服务，执行持久化操作。

这里主要展开的内容是 RDB 持久化操作的写文件过程，读过程和写过程相反。子进程的产生发生在 rdbSaveBackground\(\) 中，真正的 RDB 持久化操作是在 rdbSave\(\)，想要直接进行 RDB 持久化，调用 rdbSave\(\) 即可。



## AOF 持久化运作机制\(日志模式\) {#71d9a05d4b6f801fa6e04ab051d3ff64}

和 redis RDB 持久化运作机制不同，redis AOF 有后台执行和边服务边备份两种方式。

![](http://wiki.jikexueyuan.com/project/redis/images/redis18.png)

1）AOF 后台执行的方式和 RDB 有类似的地方，fork 一个子进程，主进程仍进行服务，子进程执行AOF 持久化，数据被dump 到磁盘上。与 RDB 不同的是，后台子进程持久化过程中，主进程会记录期间的所有数据变更（主进程还在服务），并存储在 server.aof\_rewrite\_buf\_blocks 中；后台子进程结束后，Redis 更新缓存追加到 AOF 文件中，是 RDB 持久化所不具备的。

来说说更新缓存这个东西。Redis 服务器产生数据变更的时候，譬如 set name Jhon，不仅仅会修改内存数据集，也会记录此更新（修改）操作，记录的方式就是上面所说的数据组织方式。

更新缓存可以存储在 server.aof_buf 中，你可以把它理解为一个小型临时中转站，所有累积的更新缓存都会先放入这里，它会在特定时机写入文件或者插入到server.aof_-rewrite\_buf\_blocks 下链表（下面会详述）；server.aof_buf 中的数据在 propagrate\(\) 添加，在涉及数据更新的地方都会调用propagrate\(\) 以累积变更。更新缓存也可以存储在 server.aof_-rewrite\_buf\_blocks，这是一个元素类型为 struct aofrwblock 的链表，你可以把它理解为一个仓库，当后台有AOF 子进程的时候，会将累积的更新缓存（在 server.aof\_buf 中）插入到链表中，而当 AOF 子进程结束，它会被整个写入到文件。两者是有关联的。

这里的意图即是不用每次出现数据变更的时候都触发一个写操作，可以将写操作先缓存到内存中，待到合适的时机写入到磁盘，如此避免频繁的写操作。当然，完全可以实现让数据变更及时更新到磁盘中。两种做法的好坏就是一种博弈了。



Redis允许同时开启AOF和RDB，既保证了数据安全又使得进行备份等操作十分容易。此时重新启动Redis后Redis会使用AOF文件来恢复数据，因为AOF方式的持久化可能丢失的数据更少。
