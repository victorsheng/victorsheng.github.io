title: java-io-study
date: 2018-07-05 07:05:09
tags:
    - io
    - java
categories:
---

# 概述
IO(Input And Output)在编程中是一个很常见的需求，IO即意味着我们的java程序需要和"外部"进行通信，这个"外部"可以是很多介质

1. 本地磁盘文件、远程磁盘文件
2. 数据库连接
3. TCP、UDP、HTTP网络通信
4. 进程间通信
5. 硬件设备(键盘、串口等)

![upload successful](/images/pasted-205.png)


# java io体系
```
Object(超类)
1. 基于"字节"操作的 I/O 接口:
    1) InputStream
    InputStream类是一个abstract class(抽象父类)，它不能被直接用于实例化进行流操作，我们在编程中使用的是它的子类
        1.1) ByteArrayInputStream: 从字节数组(byte[])中进行以字节为单位的读取
        1.2) FileInputStream: 从文件中进行以字节为单位的读取
            1.2.1) SocketInputStream
            org.apache.commons.net.io.SocketInputStream: 封装了对Socket的字节型流式读取
        1.3) FilterInputStream: 用来"封装其它的输入流，并为它们提供额外的功能"
            1.3.1) InflaterInputStream 
            java.util.zip.InflaterInputStream: 从压缩数据源(zip)中以字节为单位读取数据
                1.3.1.1) ZipInputStream 
                java.util.zip.ZipInputStream: 从zip文件中以字节为单位读取数据
            1.3.2) BufferedInputStream: 开辟"内部字节数组"对输入流进行缓存，函数的返回也是一个字节数组
            1.3.3) DataInputStream:
            DataInputStream 是用来装饰其它输入流，它"允许应用程序以与机器无关方式从底层输入流中读取基本 Java 数据类型"。应用程序可以使用DataOutputStream(数据输出流)
　　　　　　　写入由DataInputStream(数据输入流)读取的数据。
        1.4) ObjectInputStream: 从输入流中读取序列化后的数据，并进行反序列化(deserializes)
        1.5) PipedInputStream: 从管道中读取数据
    2) OutputStream
    OutputStream类是一个abstract class(抽象父类)，它不能被直接用于实例化进行流操作，我们在编程中使用的是它的子类
        2.1) ByteArrayOutputStream: 以字节为单位将数据写入到从字节数组(byte[])中 
        2.2) FileOutputStream: 以字节为单位将数据写入到文件中
            2.2.1) SocketOutputStream
            org.apache.commons.net.io.SocketOutputStream: 封装了对Socket的字节型流式写入
        2.3) FilterOutputStream: 用来"封装其它的输出流，并为它们提供额外的功能"
            2.3.1) ZipOutputStream: java.util.zip.ZipOutputStream: 以字节为单位向zip文件写入数据
            2.3.2) PrintStream: 
            PrintStream 是用来装饰其它输出流。它能为其他输出流添加了功能，使它们能够方便地打印各种数据值表示形式
            2.3.3) DataOutputStream:
            DataOutputStream 是用来装饰其它输入流，它"允许应用程序以与机器无关方式向底层输出流中写入基本 Java 数据类型"。应用程序可以使用DataInputStream(数据输入流)
　　　　　　　写入由DataOutputStream(数据输出流)写入的数据()。有点类似管道、或者进程间通信的感觉
　　　　　　　2.3.4) BufferedInputStream: 
        2.4) ObjectOutputStream: 对数据进行序列化(serializes)，并向输出流中写入序列化后的数据
        2.5) PipedOutputStream: 向管道中写入数据
2. 基于"字符"操作的 I/O 接口
不管是磁盘还是网络传输，最小的存储单元都是字节，而不是字符，所以 I/O 操作的都是字节而不是字符，为了操作方便，java封装了一个直接写字符的 I/O 接口，这里就涉及到java的流机制
中的一个很重要的概念，包装(装饰)。即所有的流操作在底层实现都是字节流的形式，以这个底层字节流为基础，在其上封装了各种"附加功能"(缓存、字符、管道..)
    1) Reader
    Reader类是一个abstract class(抽象父类)，它不能被直接用于实例化进行流操作，我们在编程中使用的是它的子类
        1.1) InputStreamReader:
        我们知道，字符型的流接口是在字节型的流接口基础之上进行了一次封装，提供了一些额外的功能。所以，从名字上也可以看出来，InputStreamReader是字节流通向字符流的桥梁,　　　　 
　　　　 封裝了InputStream在里头, 它以较高级的方式,一次读取一个一个字符，以文本格式输入/输出，可以指定编码格式。
            1.1.1) FileReader: 提供对文本文件(保存字符的文件)进行以字符为单位的读取
        1.2) BufferedReader:
        BufferedReader会一次性从物理流中读取8k(默认数值,可以设置)字节内容到内存，如果外界有请求，就会到这里存取，如果内存里没有才到物理流里再去读。即使读，也是再8k
　　　　 而直接读物理流，是按字节来读。对物理流的每次读取，都有IO操作。IO操作是最耗费时间的。BufferedReader就是减少了大量IO操作，节省了时间
        1.3) CharArrayReader:
        CharArrayReader 是字符数组输入流。它和ByteArrayInputStream类似，只不过ByteArrayInputStream是字节数组输入流，而CharArray是字符数组输入流。
　　　　 CharArrayReader 是用于读取字符数组，它继承于Reader。操作的数据是以字符为单位
        1.4) FilterReader: 用来"封装其它的字符输入流，并为它们提供额外的功能"
        1.5) PipedReader: PipedReader 是字符管道输入流，它继承于Reader。
        1.6) StringReader: 以String作为数据源，进行以字符为单位的读取
　　    2) Writer
   　　 Writer类是一个abstract class(抽象父类)，它不能被直接用于实例化进行流操作，我们在编程中使用的是它的子类
        2.1) OutputStreamWriter:
            2.1.1) FileWriter: 提供对文本文件(保存字符的文件)进行以字符为单位的写入
        2.2) BufferedWriter
        2.3) StringWriter
        2.4) PipedWriter
        2.5) PrintWriter 
        2.6) CharArrayWriter
3. 基于"磁盘"操作的 I/O 接口:
    1) File: (文件特征与管理): 用于文件或者目录的描述信息，例如生成新目录，修改文件名，删除文件，判断文件所在路径等，它不负责数据的输入输出，而专门用来管理磁盘文件与目录
        1) public boolean exists( )        判断文件或目录是否存在
        2) public boolean isFile( )        判断是文件还是目录 
        3) public boolean isDirectory( )    判断是文件还是目录
        4) public String getName( )        返回文件名或目录名
        5) public String getPath( )        返回文件或目录的路径。
        6) public long length( )        获取文件的长度 
        7) public String[] list( )        将目录中所有文件名保存在字符串数组中返回  
        8) public boolean renameTo( File newFile );    重命名文件
        9) public void delete( );        删除文件
        10) public boolean mkdir( );        创建目录
4. 基于网络操作的 I/O 接口:
    1) Socket
```


# 参考
http://www.cnblogs.com/LittleHann/p/3678685.html