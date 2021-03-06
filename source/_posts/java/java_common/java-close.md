---
title: java-close
abbrlink: 1203295839
date: 2018-06-25 10:03:18
tags:
categories:
---
# 避免资源泄漏
实现接口 java.io.Closeable（自 JDK 1.5）和接口 java.lang.AutoCloseable（自 JDK 1.7）的类被视为代表外部资源，当不再需要这些资源时，应该使用方法 close() 将其关闭。

Eclipse Java 编译器能够分析代码是否使用此类类型来遵从此策略。即：以下片段代表明显的资源泄漏：

    int len(File f) throws IOException {
        InputStream stream = new FileInputStream(f);
        return stream.available();
    }
编译器将使用“资源泄漏：从未关闭‘流’”来对此情况进行标记。


# 基本原则: 谁生产谁销毁

这个是用来解决责任权的问题，例如你的方法接收一个InputStream作为参数，通常就不应该在方法内去关闭它，而由客户端代码去处理。如果要关闭，通常应该在方法签名上明确说明，具体样例参考commons-io的IOUtils类。

还有另外一个例子，就是经常使用的Servlet的输入输出流，根据这个原则，也是不应该在代码中进行关闭的，这个工作是由Web容器负责的。


# 关闭的是什么?
java本身是带GC的，所以对象在消除引用之后，按正常是能够被回收的，那么为什么会有关闭操作?

这是为了回收系统资源，主要是端口(网络IO),文件句柄(输入输出)等，通常涉及的场景也是操作文件，网络操作、数据库应用等。对于类unix系统，所有东西都是抽象成文件的，所以可以通过lsof来观察。

```
lsof -p 25945 |grep /tmp/data | wc -l
88

ls  /proc/25945/fd | wc -l
93
```
如果有关闭操作的话，就会发现打开文件数一直都处于很低的位置。如果持续出现未关闭的情况，积累到一定程度就可能超过系统限制，出现too many open files的错误。

# 文件句柄
在linux中, 一切皆句柄, 比如file, socket都是, 每个进程都有自己的上限, 一旦超过这个上限,系统就会限制并拒绝相应的资源请求. 
查询命令
```
ulimit -a  
ulimit -a -H


cd /proc/9082/fd
lsof -p 9082
```



# 资源包装程序和可用资源可关闭
JDK 定义实现可关闭的一些类，而不是直接在操作系统级别代表资源。

java.io.StringReader 是一个不需要调用 close() 的可关闭示例，因为，不保留需要清理的任何操作系统资源。该分析使用白色列表从归到此类别的 java.io 检测类。没有针对这些类发出任何资源泄漏警告。

类似 java.io.BufferedInputStream 的类的实例是围绕另一资源的包装程序（在此可在多个级别应用包装程序）。另外，这些对象不直接代表操作系统资源。如果关闭了合并的资源，那么不需要关闭包装程序。相反，如果关闭了包装程序，那么这将包括合并资源的关闭。该分析具有用于检测包装程序资源的第二个白色列表，并将识别底层实际资源是否将通过包装程序直接或间接关闭。任何一个都满足针对资源泄漏的静默警告。白列表包含来自 java.io、java.util.zip、java.security、java.beans 和 java.sound.sampled 的类。

提示：关闭最外层的包装程序（而不是合并的资源）通常是更可取/最安全的。