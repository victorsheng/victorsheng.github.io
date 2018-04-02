---
title: source-aqs
date: 2018-04-02 12:38:41
tags:
categories:
---
# 背景和简介
AbstractQueuedSynchronizer简称AQS是一个抽象同步框架，可以用来实现一个依赖状态的同步器。
JDK1.5中提供的java.util.concurrent包中的大多数的同步器(Synchronizer)如Lock, Semaphore, Latch, Barrier等，这些类之间大多可以互相实现，如使用Lock实现一个Semaphore或者反过来，但是它们都是基于java.util.concurrent.locks.AbstractQueuedSynchronizer这个类的框架实现的，理解了这个稍微复杂抽象的类再去理解其他的同步器就很轻松了。

等待条件发生的操作是开发中常见的需求，例如实现一个生产者消费者，当队列为空时，消费者获取消息可以立即返回并不断轮询，另外一种方式是阻塞直到有新的消息，后者对调用方更加友好一些。

# 设计与现实

要完成这些，可以抽象出一些基本功能。
首先，需要保存状态提供给子类去表示具体的含义，并且提供查询、设置、原子修改的操作。
其次，要能够阻塞、唤醒线程。
然后，还需要有一个容器来存放等待的线程

## 初步设计
AQS内部通过一个int类型的state字段表示同步状态，状态的具体含义可以子类来定义，例如ReentrantLock中用state表示线程重入的次数，Semaphore表示可用的许可的数量等。使用int是由于int能够应对大部分的场景，而long在很多平台需要使用额外锁来保证一致性的读取。

类似模板模式，AQS暴露给子类一些方法实现(如tryAcquire，tryRelease), 获取操作通常是依赖state状态的，当状态允许时获取成功否则加入等待队列一直等待到允许的状态发生时重新获取，例如，Semaphore中当state(即许可数量)小于等于0时，获取不能成功。释放操作通过会修改状态并判断是否能让其他等待的线程能够重新获取，例如ReentrantLock中释放会减少重入次数并检查重入次数是否达到了0，达到0说明已经解锁，这时会通知其他等待线程重新获取。所以AQS的acquire和release会通过组合应用到不同的同步器实现中实现不同的语义，如

- Reentrant.lock
- ReentrantReadWriteLock
- Semaphore.acquire,
- CountDownLatch.await
- SynchronousQueue
- FutureTask.get(1.5以后不再使用AQS)


等。
另外AQS还提供其他功能，如非阻塞的获取tryAcquire, 带有超时时间的获取，可以中断的获取等。
AQS可以根据具体的场景提供exclusive模式和shared模式，在exclusive模式下，同一时刻最多只能有一个线程能够处于成功获取的状态，排他锁是一个exclusive模式的例子，shared模式则可以多个线程一起获取成功，如多个许可的Semaphore。
另外在java.util.concurrent包中还定义了Condition接口，用来提供监视器风格的等待通知操作，可以替换Object中基于synchronized、监视器锁的wait和notify机制。

实现具体的同步器时，需要将实现类作为具体同步器的内部类，然后调用实现类的acquire、acquireShared、release等方法。

## 更具体的设计

### 同步状态synchronization管理(state字段)
state字段可以通过volatile修饰，这样get和set方法具有了voaltile的相关语义，通过可以通过AtomicInteger或Unsafe类来实现原子CAS更新操作，基于减少indirection的考虑，JDK中一般都是使用Unsafe类。


### 阻塞、唤醒线程
除了基于内置的监视器的synchronizer机制外，唯一可用的似乎是已经Deprecated的Thread.stop、Thread.suspend、Thread.resume, 而这几个个方法有严重的安全问题比如可能造成死锁。JDK1.5增加了一个LockSupport类来解决这个问题, LockSupport给每个线程绑定一个类似Semaphore的permit许可数量，不过permit最大为1，初始时permit为0。park()操作阻塞一直到permit为1并且将permit减1。unpark则进行加1，不过unpark并不会累加permit。
而且如果先调用unpark()的话，之后调用park()会立即返回。
park()返回的几个情况是

之前有其他线程调用剩余的unpark()或在park()之后其他调用了unpark()
其他线程中断了目标线程
调用虚假返回（类似Object.wait()的伪通知, 所以调用返回时需要检查是否等待条件已经达成
hotspot的LockSupport最终还是使用了Unsafe的功能

###  线程等待队列
AQS的核心是一个线程等待队列，采用的是一个先进先出FIFO队列。用来实现一个非阻塞的同步器队列有主要有两个选择Mellor-Crummey and Scott (MCS) locks和Craig, Landin, and Hagersten (CLH) locks的变种。CLH锁更适合处理取消和超时，所以AQS基于CLH进行修改作为线程等待队列。
CLH队列使用pred引用前节点形成一个队列，入队enqueue和出队dequeue操作都可以通过原子操作完成。

CLH队列有很多优点，包括入队和出队非常快、无锁、非阻塞，并且是否有线程等待只需要判断head和tail是否相等。
CLH中并没有只有向后的指针引用后继节点，每个节点只需要修改自己的状态就能通知后继节点。但是在AQS这样的阻塞同步器中，需要主动唤醒(unpark)后继节点。所以在AQS中增加了next引用引用后继节点，但是并没有合适的方法使用CAS进行无锁插入双向链表的方法，所以节点的插入并不是原子完成的，需要在CAS替换tail完成后调用pred.next=node。


### ConditionObject
AQS还提供了一个ConditionObject类来表示监视器风格的等待通知操作，主要用在Lock中，和传统的监视器的区别在于一个Lock可以创建多个Condition。ConditionObject使用相同的内部队列节点，但是维护在一个单独的条件队列中，当收到signal操作的时候将条件队列的节点转移到等待队列。


# 代码实现

## 节点(5个字段)
```
static final class Node {

	// 省略部分代码...
		
	volatile Node prev; // 前节点
	
	volatile Node next; // 后节点

	volatile int waitStatus;// 等待状态
	
	volatile Thread thread; // 节点线程
	
	Node nextWaiter; // 节点模式

	Node() { }
	
	Node(Thread thread, Node mode) { 
		this.nextWaiter = mode;
		this.thread = thread;
	}
	
	Node(Thread thread, int waitStatus) { 
		this.waitStatus = waitStatus;
		this.thread = thread;
	}
}
```
### 节点状态
```
// 表示当前节点的后续节点中的线程通过 park 被阻塞了，需要通过unpark解除它的阻塞
static final int SIGNAL = -1;

// 表示当前节点在 condition 队列中
static final int CONDITION = -2;

// 共享模式的头结点可能处于此状态，表示无条件往下传播
// 引入此状态是为了优化锁竞争，使队列中线程有序地一个一个唤醒
static final int PROPAGATE = -3;

//表示当前节点的线程因为超时或中断被取消了
static final int CANCELLED = 1;

// 关键 -> 0 表示初始化状态
```
Node 的等待状态，因为大于 0 的只有 CANCELLED 一种状态。因此在很多地方也用 waitStatus > 0 表示该状态

## CLH 队列(基本可以保持公平)
- 在 JUC 框架中，若有线程尝试获取锁时失败时，该线程会被包装成节点添加进此队列，也称入队。
- 在 JUC 框架中，若有线程释放锁时，等待队列的头节点会从队列中移除，表示不用再等待，也称出队。

### 入队
在 AQS 中通过 addWaiter 方法将节点加入等待队列：
```
// 头节点
private transient volatile Node head;

// 尾节点
private transient volatile Node tail;

private Node addWaiter(Node mode) {

	Node node = new Node(Thread.currentThread(), mode);
	
	Node pred = tail;

	// 为空表示等待队列为空
	if (pred != null) {
		node.prev = pred;
		// 通过 CAS 操作设置 tail 为 node
		// 关键 ->失败表示在这期间有其他线程的节点被设置为新的尾节点
		if (compareAndSetTail(pred, node)) {
			pred.next = node;
			return node;
		}
	}
	
	// 关键 -> 当[等待队列]为空，或者新节点入队失败时（说明存在并发），代码才会执行到这
	enq(node);
	
	return node;
}

// 往等待队列中（尾部）插入一个节点
private Node enq(final Node node) {
	// 关键 -> 自旋直至成功
	for (;;) {
		Node t = tail;

		// t 为空表示等待队列为空
		if (t == null) {
			// 构建等待队列的头节点，实质是创建一个空的循环双链表
			if (compareAndSetHead(new Node())){
				tail = head;
			}
		} else {
			// 设置该节点为新的尾节点
			node.prev = t;
			if (compareAndSetTail(t, node)) {
				t.next = node;
				return t;
			}
		}
	}
}
```
入队操作只与队列的尾节点有关，通过原子操作将新节点设置成新的尾节点。若操作失败则通过不断自旋直至成功。

### 出队
出队操作跟头节点有关，将要执行出队的节点设置为新的头节点，并置空旧的头节点从等待队列移除即可。

- 设置新的 head
- 清除节点上的线程
- 将旧的头节点从等待队列移除（ 断开前指针和后指针）

```
if (p == head && tryAcquire(arg)) {
	// 设置新的头节点
    setHead(node);
	
	// 将旧头节点后指针置空，表示从等待队列移除
    p.next = null; 
    failed = false;
    return interrupted;
}

private void setHead(Node node) {
	head = node;
	node.thread = null;
	node.prev = null;
}
```

## 共享模式
共享式的操作与独占式的主要区别在于，每次acquire竞争失败后，独占式将立即阻塞当前线程，而共享式需要在多次acquire失败后才会阻塞当前线程。简单来说，共享式是有一定限额的独占式。限额的满足方式，根据不同的子类不同的实现方式。