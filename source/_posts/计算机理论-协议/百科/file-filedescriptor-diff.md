title: file-file_descriptor-diff
abbrlink: 3123063586
date: 2018-11-30 16:26:07
tags:
categories:
---
# FileDescriptor(进程级别的)

文件描述符在形式上是一个非负整数。实际上，它是一个索引值，指向内核为每一个**进程**所维护的该进程打开文件的记录表。当**进程**打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。在程序设计中，一些涉及底层的程序编写往往会围绕着文件描述符展开。但是文件描述符这一概念往往只适用于UNIX、Linux这样的操作系统。

**每个Unix进程（除了可能的守护进程）应均有三个标准的POSIX文件描述符，对应于三个标准流：**

整数值	名称	<unistd.h>符号常量[1]	<stdio.h>文件流[2]
0	Standard input	STDIN_FILENO	stdin
1	Standard output	STDOUT_FILENO	stdout
2	Standard error	STDERR_FILENO	stderr

# File
File 是“文件”和“目录路径名”的抽象表示形式。
File 直接继承于Object，实现了Serializable接口和Comparable接口。实现Serializable接口，意味着File对象支持序列化操作。
而实现Comparable接口，意味着File对象之间可以比较大小；File能直接被存储在有序集合(如TreeSet、TreeMap中)。

## nio拆分成:Path(路径)与Files(文件内容)
java.nio.file包定义了Java虚拟机访问文件，文件属性和文件系统的接口和类。 该API可能用于克服java.io.File类的许多java.io.File 。 该toPath方法可以被用于获得Path使用由A表示的抽象路径File对象查找文件。 所得到的Path可以与Files类一起使用，以提供更有效和更广泛的访问附加文件操作，文件属性和I / O异常，以帮助在文件操作失败时诊断错误。

# FileDescriptor+File与输入输出流
```
    /**
     * Creates a <code>FileInputStream</code> by
     * opening a connection to an actual file,
     * the file named by the <code>File</code>
     * object <code>file</code> in the file system.
     * A new <code>FileDescriptor</code> object
     * is created to represent this file connection.
     * <p>
     * First, if there is a security manager,
     * its <code>checkRead</code> method  is called
     * with the path represented by the <code>file</code>
     * argument as its argument.
     * <p>
     * If the named file does not exist, is a directory rather than a regular
     * file, or for some other reason cannot be opened for reading then a
     * <code>FileNotFoundException</code> is thrown.
     *
     * @param      file   the file to be opened for reading.
     * @exception  FileNotFoundException  if the file does not exist,
     *                   is a directory rather than a regular file,
     *                   or for some other reason cannot be opened for
     *                   reading.
     * @exception  SecurityException      if a security manager exists and its
     *               <code>checkRead</code> method denies read access to the file.
     * @see        java.io.File#getPath()
     * @see        java.lang.SecurityManager#checkRead(java.lang.String)
     */
    public FileInputStream(File file) throws FileNotFoundException {
        String name = (file != null ? file.getPath() : null);
        SecurityManager security = System.getSecurityManager();
        if (security != null) {
            security.checkRead(name);
        }
        if (name == null) {
            throw new NullPointerException();
        }
        if (file.isInvalid()) {
            throw new FileNotFoundException("Invalid file path");
        }
        fd = new FileDescriptor();
        fd.attach(this);
        path = name;
        open(name);
    }



     /**
     * Creates a file output stream to write to the file represented by
     * the specified <code>File</code> object. If the second argument is
     * <code>true</code>, then bytes will be written to the end of the file
     * rather than the beginning. A new <code>FileDescriptor</code> object is
     * created to represent this file connection.
     * <p>
     * First, if there is a security manager, its <code>checkWrite</code>
     * method is called with the path represented by the <code>file</code>
     * argument as its argument.
     * <p>
     * If the file exists but is a directory rather than a regular file, does
     * not exist but cannot be created, or cannot be opened for any other
     * reason then a <code>FileNotFoundException</code> is thrown.
     *
     * @param      file               the file to be opened for writing.
     * @param     append      if <code>true</code>, then bytes will be written
     *                   to the end of the file rather than the beginning
     * @exception  FileNotFoundException  if the file exists but is a directory
     *                   rather than a regular file, does not exist but cannot
     *                   be created, or cannot be opened for any other reason
     * @exception  SecurityException  if a security manager exists and its
     *               <code>checkWrite</code> method denies write access
     *               to the file.
     * @see        java.io.File#getPath()
     * @see        java.lang.SecurityException
     * @see        java.lang.SecurityManager#checkWrite(java.lang.String)
     * @since 1.4
     */
    public FileOutputStream(File file, boolean append)
        throws FileNotFoundException
    {
        String name = (file != null ? file.getPath() : null);
        SecurityManager security = System.getSecurityManager();
        if (security != null) {
            security.checkWrite(name);
        }
        if (name == null) {
            throw new NullPointerException();
        }
        if (file.isInvalid()) {
            throw new FileNotFoundException("Invalid file path");
        }
        this.fd = new FileDescriptor();
        fd.attach(this);
        this.append = append;
        this.path = name;

        open(name, append);
    }
```


# 输出java进程的文件描述符

```
Mac-Pro:tmp demo$ lsof -p 50581
COMMAND   PID   USER   FD   TYPE             DEVICE   SIZE/OFF     NODE NAME
java    50581 demo  cwd    DIR                1,5        704  1589089 /Users/demo/code/vicProjects/demo
java    50581 demo  txt    REG                1,5      93520 12252268 /Library/Java/JavaVirtualMachines/jdk-11.0.1.jdk/Contents/Home/bin/java
java    50581 demo  txt    REG                1,5      42000 12252355 /Library/Java/JavaVirtualMachines/jdk-11.0.1.jdk/Contents/Home/lib/libverify.dylib
java    50581 demo  txt    REG                1,5     182312 12252363 /Library/Java/JavaVirtualMachines/jdk-11.0.1.jdk/Contents/Home/lib/libjava.dylib
java    50581 demo  txt    REG                1,5     223112 12252329 /Library/Java/JavaVirtualMachines/jdk-11.0.1.jdk/Contents/Home/lib/libjdwp.dylib
java    50581 demo  txt    REG                1,5   13843396 12252333 /Library/Java/JavaVirtualMachines/jdk-11.0.1.jdk/Contents/Home/lib/server/libjvm.dylib
java    50581 demo  txt    REG                1,5      32768 14235308 /private/var/folders/ky/bxkqd1m55zj3zybvmjwrrxdw0000gn/T/hsperfdata_demo/50581
java    50581 demo  txt    REG                1,5      31056 12252304 /Library/Java/JavaVirtualMachines/jdk-11.0.1.jdk/Contents/Home/lib/libzip.dylib
java    50581 demo  txt    REG                1,5      27816 12252313 /Library/Java/JavaVirtualMachines/jdk-11.0.1.jdk/Contents/Home/lib/libjimage.dylib
java    50581 demo  txt    REG                1,5      21200 12252310 /Library/Java/JavaVirtualMachines/jdk-11.0.1.jdk/Contents/Home/lib/libdt_socket.dylib
java    50581 demo  txt    REG                1,5      53836 12252302 /Library/Java/JavaVirtualMachines/jdk-11.0.1.jdk/Contents/Home/lib/libnio.dylib
java    50581 demo  txt    REG                1,5      79256 12252301 /Library/Java/JavaVirtualMachines/jdk-11.0.1.jdk/Contents/Home/lib/libnet.dylib
java    50581 demo  txt    REG                1,5     841456 10116948 /usr/lib/dyld
java    50581 demo  txt    REG                1,5  137462057 12252359 /Library/Java/JavaVirtualMachines/jdk-11.0.1.jdk/Contents/Home/lib/modules
java    50581 demo  txt    REG                1,5 1170472960 10131702 /private/var/db/dyld/dyld_shared_cache_x86_64h
java    50581 demo    0   PIPE 0x7e75311251aa53b5      16384          ->0x7e75311251aa73f5
java    50581 demo    1   PIPE 0x7e75311251aa65b5      16384          ->0x7e75311251aa5d75
java    50581 demo    2   PIPE 0x7e75311251aa4e75      16384          ->0x7e75311251aa64f5
java    50581 demo    3r   REG                1,5  137462057 12252359 /Library/Java/JavaVirtualMachines/jdk-11.0.1.jdk/Contents/Home/lib/modules
java    50581 demo    4u  IPv4 0x7e75311287cb061d        0t0      TCP localhost:58934->localhost:58933 (ESTABLISHED)
java    50581 demo    5r   REG                1,5          4 14235293 /Users/demo/tmp/qingshu.txt
java    50581 demo    6r   REG                1,5          4 14235293 /Users/demo/tmp/qingshu.txt
java    50581 demo    7r   REG                1,5          4 14235293 /Users/demo/tmp/qingshu.txt
java    50581 demo    8r   REG                1,5          4 14235293 /Users/demo/tmp/qingshu.txt
java    50581 demo    9r   REG                1,5          4 14235293 /Users/demo/tmp/qingshu.txt
java    50581 demo   10r   REG                1,5          4 14235293 /Users/demo/tmp/qingshu.txt
java    50581 demo   11r   REG                1,5          4 14235293 /Users/demo/tmp/qingshu.txt
java    50581 demo   12r   REG                1,5          4 14235293 /Users/demo/tmp/qingshu.txt
java    50581 demo   13r   REG                1,5          4 14235293 /Users/demo/tmp/qingshu.txt
java    50581 demo   14r   REG                1,5          4 14235293 /Users/demo/tmp/qingshu.txt
```
首先是0,1,2
然后是3r,4u,5r,6r........

# COMMAND：进程的名称
PID：进程标识符
USER：进程所有者
FD：文件描述符，应用程序通过文件描述符识别该文件。如cwd、txt等
TYPE：文件类型，如DIR、REG等
DEVICE：指定磁盘的名称
SIZE：文件的大小
NODE：索引节点（文件在磁盘上的标识）
NAME：打开文件的确切名称

# 三个表
- 进程级的文件描述符表
- 系统级的文件描述符表
- 文件系统的i-node表


![upload successful](/images/pasted-298.png)

## 1.进程级的描述符表
进程级的描述符表的每一条目记录了单个文件描述符的相关信息。
    1. 控制文件描述符操作的一组标志。（目前，此类标志仅定义了一个，即close-on-exec标志）
    2. 对打开文件句柄的引用

## 2.系统级的描述符表格（open file description table）or 打开文件表（open file table）
内核对所有打开的文件的文件维护有一个系统级的描述符表格（open file description table）。有时，也称之为打开文件表（open file table），并将表格中各条目称为打开文件句柄（open file handle）。一个打开文件句柄存储了与一个打开文件相关的全部信息，如下所示：
    1. 当前文件偏移量（调用read()和write()时更新，或使用lseek()直接修改）
    2. 打开文件时所使用的状态标识（即，open()的flags参数）
    3. 文件访问模式（如调用open()时所设置的只读模式、只写模式或读写模式）
    4. 与信号驱动相关的设置
    5. 对该文件i-node对象的引用
    6. 文件类型（例如：常规文件、套接字或FIFO）和访问权限
    7. 一个指针，指向该文件所持有的锁列表
    8. 文件的各种属性，包括文件大小以及与不同类型操作相关的时间戳

## 3.inode
文件数据都储存在"块"中，那么很显然，我们还必须找到一个地方储存文件的元信息，比如文件的创建者、文件的创建日期、文件的大小等等。这种储存文件元信息的区域就叫做inode，中文译名为"索引节点"。每一个文件都有对应的inode，里面包含了与该文件有关的一些信息。


inode包含文件的元信息，具体来说有以下内容：
　　* 文件的字节数
　　* 文件拥有者的User ID
　　* 文件的Group ID
　　* 文件的读、写、执行权限
　　* 文件的时间戳，共有三个：ctime指inode上一次变动的时间，mtime指文件内容上一次变动的时间，atime指文件上一次打开的时间。
　　* 链接数，即有多少文件名指向这个inode
　　* 文件数据block的位置

# 参考
https://zh.wikipedia.org/wiki/%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6
https://segmentfault.com/a/1190000009724931
http://www.ruanyifeng.com/blog/2011/12/inode.html