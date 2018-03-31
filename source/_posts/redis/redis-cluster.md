title: redis集群
date: 2018-03-28 22:33:59
tags:
categories:
---
https://blog.csdn.net/u011204847/article/details/51307044
http://www.redis.cn/topics/cluster-tutorial.html
https://www.jianshu.com/p/04dd90ea08f5

# 简介
Redis Cluster是Redis官方在Redis 3.0版本正式推出的高可用以及分布式的解决方案。

Redis Cluster由多个Redis实例组成的整体，数据按照槽(slot)存储分布在多个Redis实例上，通过Gossip协议来进行节点之间通信。

Redis Cluster实现的功能：

• 将数据分片到多个实例(按照slot存储)；

• 集群节点宕掉会自动failover；

• 提供相对平滑扩容(缩容)节点。



Redis Cluster暂未有的：

• 实时同步

• 强一致性

--------

redis集群是一个无中心的分布式Redis存储架构，可以在多个节点之间进行数据共享，解决了Redis高可用、可扩展等问题。redis集群提供了以下两个好处

1、将数据自动切分(split)到多个节点
2、当集群中的某一个节点故障时，redis还可以继续处理客户端的请求。

一个 Redis 集群包含 16384 个哈希槽（hash slot），数据库中的每个数据都属于这16384个哈希槽中的一个。集群使用公式 CRC16(key) % 16384 来计算键 key 属于哪个槽。集群中的每一个节点负责处理一部分哈希槽。



## Redis 集群的数据分片
Redis 集群没有使用一致性hash, 而是引入了 哈希槽的概念.

Redis 集群有16384个哈希槽,每个key通过CRC16校验后对16384取模来决定放置哪个槽.集群的每个节点负责一部分hash槽,举个例子,比如当前集群有3个节点,那么:

节点 A 包含 0 到 5500号哈希槽.
节点 B 包含5501 到 11000 号哈希槽.
节点 C 包含11001 到 16384号哈希槽.
这种结构很容易添加或者删除节点. 比如如果我想新添加个节点D, 我需要从节点 A, B, C中得部分槽到D上. 如果我想移除节点A,需要将A中的槽移到B和C节点上,然后将没有任何槽的A节点从集群中移除即可. 由于从一个节点将哈希槽移动到另一个节点并不会停止服务,所以无论添加删除或者改变某个节点的哈希槽的数量都不会造成集群不可用的状态.

何处维护hash槽的绑定关系


## cap理论
Redis Cluster集群功能推出已经有一段时间了。在单机版的Redis中，每个Master之间是没有任何通信的，所以我们一般在Jedis客户端或者Codis这样的代理中做Pre-sharding。按照CAP理论来说，单机版的Redis属于保证CP(Consistency & Partition-Tolerancy)而牺牲A(Availability)，也就说Redis能够保证所有用户看到相同的数据（一致性，因为Redis不自动冗余数据）和网络通信出问题时，暂时隔离开的子系统能继续运行（分区容忍性，因为Master之间没有直接关系，不需要通信），但是不保证某些结点故障时，所有请求都能被响应（可用性，某个Master结点挂了的话，那么它上面分片的数据就无法访问了）。

有了Cluster功能后，Redis从一个单纯的NoSQL内存数据库变成了分布式NoSQL数据库，CAP模型也从CP变成了AP。也就是说，通过自动分片和冗余数据，Redis具有了真正的分布式能力，某个结点挂了的话，因为数据在其他结点上有备份，所以其他结点顶上来就可以继续提供服务，保证了Availability。然而，也正因为这一点，Redis无法保证曾经的强一致性了。这也是CAP理论要求的，三者只能取其二。


# 集群中的主从复制

集群中的每个节点都有1个至N个复制品，其中一个为主节点，其余的为从节点，如果主节点下线了，集群就会把这个主节点的一个从节点设置为新的主节点，继续工作。这样集群就不会因为一个主节点的下线而无法正常工作。

注意：

1、如果某一个主节点和他所有的从节点都下线的话，redis集群就会停止工作了。redis集群不保证数据的强一致性，在特定的情况下，redis集群会丢失已经被执行过的写命令

2、使用异步复制（asynchronous replication）是 Redis 集群可能会丢失写命令的其中一个原因，有时候由于网络原因，如果网络断开时间太长，redis集群就会启用新的主节点，之前发给主节点的数据就会丢失。


# Redis Cluster分片实现
一般分片（Sharding）实现的方式有list、range和hash(或者基于上述的组合方式)等方式。而Redis的实现方式是基于hash的分片方式，具体是虚拟槽分区。

虚拟槽分区

槽(slot):使用分散度良好的hash函数把所有数据映射到一个固定范围的整数集合中，这个整数集合就是槽。

Redis Cluster槽: Redis Cluster槽的范围是0 ~ 16383。槽是集群内数据管理和迁移的基本单位。

![upload successful](/images/pasted-108.png)


## Hash标签

哈希标签(hash tags)，在Redis集群分片中，可以通过哈希标签来实现指定两个及以上的Key在同一个slot中。只要Key包含“{…}”这种模式，Redis就会根据第一次出现的’{’和第一次出现的’}’之间的字符串进行哈希计算以获取相对应的slot数。如上Redis源码实现。

所以如果要指定某些Key存储到同一个slot中，只需要在命令Key的之后指定相同的“{…}”命名模式即可。



## 集群节点和槽
当Redis Cluster中的16384个槽都有节点在处理时，集群处于上线状态(ok)；

如果Redis Cluster中有任何一个槽没有得到处理(或者某一分片的最后一个节点挂了)，那么集群处于下线状态(fail)。（info cluster中的：cluster_state状态）。那整个集群就不能对外提供服务。

Redis-3.0.0.rc1加入cluster-require-full-coverage参数，默认关闭，打开集群容忍部分失败。

但是如果集群超过半数以上master挂掉，无论是否有slave集群进入fail状态。

## 节点ID



Redis Cluster每个节点在集群中都有唯一的ID，该ID是由40位的16进制字符组成，具体是节点第一次启动由linux的/dev/urandom生成。具体信息会保存在node.cnf配置文件中(该文件有Redis Cluster自动维护，可以通过参数cluster-config-file来指定路径和名称)，如果该文件被删除，节点ID将会重新生成。（删除以后所有的cluster和replication信息都没有了）或者通过Cluster Reset强制请求硬重置。

节点ID用于标识集群中的每个节点，包括指定Replication Master。只要节点ID不改变，哪怕节点的IP和端口发生了改变，Redis Cluster可以自动识别出IP和端口的变化，并将变更的信息通过Gossip协议广播给其他节点。


## ClusterNode
![upload successful](/images/pasted-109.png)

Master 节点维护这一个16384/8字节的位序列，Master节点用bit来标识对于某个槽自己是否拥有。(判断索引是不是为1即可)

slots属性是一个二进制位数组(bit arry)，这个数组的长度为16384/8 = 2048个字节，共包含16384个二进制。

Redis Cluster对slots数组中的16384个二进制位进行编号：从0为起始索引，16383为终止索引。

根据索引i上的二进制位的值来判断节点是否负责处理槽i：

•slots数组在索引i上的二进制位的值为1，即表示该节点负责处理槽i；

•slots数组在索引i上的二进制位的值为0，即表示该节点不负责处理槽i；

示例1：（如下节点负责处理slot0-slot7）

![upload successful](/images/pasted-110.png)
即在Redis Cluster中Master节点使用bit(0)和bit(1)来标识对某个槽是否拥有，而Master只要判断序列第二位的值是不是1即可，时间复杂度为O(1)。

numslots属性记录节点负责处理的槽的数量，也就是slots数组中值为1的二进制位的数量。上图中节点处理的槽数量为8个。

## ClusterState
集群中所有槽的分配信息都保存在ClusterState数据结构的slots数组中，程序要检查槽i是否已经被分配或者找出处理槽i的节点，只需要访问clusterState.slots[i]的值即可，时间复杂度为O(1)。

slots数组包含16384个项，每个数组项都是一个指向clusterNode结构的指针：

•如果slots[i]指针指向null，那么表示槽i尚未指派给任何节点；

•如果slots[i]指针指向一个clusterNode结构，那么表示槽i已经指派给了clusterNode结构所代表的节点。

![upload successful](/images/pasted-111.png)
示例2：

1.  slots[0]至slots[4999]的指针都指向端口为6381的节点，即槽0到4999都由节点6381负责处理；

2.  slots[5000]至slots[9999]的指针都指向端口为6382的节点，即槽5000到9999都由节点6382负责处理；

3. slots[10000]至slots[16383]的指针都指向端口为6383的节点，即槽10000到16384都由节点6383负责处理。

![upload successful](/images/pasted-112.png)

## 数组 clusterNode.slots和clusterState.slots：

• clusterNode.slots数组记录了clusterNode结构所代表的节点的槽指派信息（每个节点负责哪些槽）。

• clusterState.slots数组记录了集群中所有槽的指派信息。

• 如果需要查看某个节点的槽指派信息，只需要将相应节点的clusterNode.slots数组整个发送出去即可。

• 但是如果需要查看槽i是否被分配或者分配给了哪个节点，就需要遍历clusterState.nodes字典中所有clusterNode结构，检查这些结构的slots数组，直到遍历到负责处理槽i的节点为止，这个过程的时间复杂度为O(N)，N是clusterState.nodes字典保存的clusterNode结构的数量。

• 引入clusterState.slots ,将所有槽的指派信息保存在clusterState.slots数组里面，程序要检查槽i是否已经被指派，或者查看负责处理槽i的节点，只需要访问clusterState.slots[i]的值即可，这个操作的时间复杂度为O(1)。

• 如果只使用clusterState.slots数组(不引入clusterNode.slots)，如果要将节点A的槽指派信息传播给其他节点时，必须先遍历整个clusterState.slots数组，记录节点A负责处理哪些槽，然后再发送给其他节点。比直接发送clusterNode.slots数组要低效的多。

## Redis Cluster节点通信

Redis Cluster节点通信
**Redis Cluster采用P2P的Gossip协议，Gossip协议的原理就是每个节点与其他节点间不断通信交换信息，一段时间后节点信息一致，每个节点都知道集群的完整信息**。

Redis Cluster通信过程：

(1)集群中的每个节点都会单独开辟一个TCP通道，用于节点之间彼此通信，通信端口号在基础端口上加10000；

(2)每个节点在固定周期内通过特定规则选择几个节点发送ping消息；

(3)接收到ping消息的节点用pong消息作为响应。

集群中每个节点通过一定规则挑选要通信的节点，每个节点可能知道全部节点，也可能仅知道部分节点，

只要这些节点彼此可以正常通信，最终它们会达到一致的状态。当节点出故障、新节点加入、主从角色变化、槽信息变更等事件发生时，通过不断的ping/pong消息通信，经过一段时间后所有的节点都会知道整个集群全部节点的最新状态，从而达到集群状态同步的目的。



## Gossip消息

Gossip协议的主要职责就是信息交换，信息交换的载体就是节点彼此发送的Gossip消息，常用的Gossip消息可分为：

• meet消息：用于通知新节点加入。消息发送者通知接收者加入到当前集群，meet消息通信正常完成后，接收节点会加入到集群中并进行周期性的ping、pong消息交换；

• ping消息：集群内交换最频繁的消息，集群内每个节点每秒向多个其他节点发送ping消息，用于检测节点是否在线和交换彼此状态信息。ping消息发送封装了自身节点和部分其他节点的状态数据。

• pong消息：当接收到ping、meet消息时，作为响应消息回复给发送方确认消息正常通信。pong消息内部封装了自身状态数据。节点也可以向集群内广播自身的pong消息来通知整个集群对自身状态进行更新。

• fail消息：当节点判定集群内另一个节点下线时，会向集群内广播一个fail消息，其他节点接收到fail消息之后把对应节点更新为下线状态。

消息格式：

所有的消息格式划分为：消息头和消息体。

消息头包含发送节点自身状态数据，接收节点根据消息头就可以获取到发送节点的相关数据。

