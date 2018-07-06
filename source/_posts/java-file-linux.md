title: 文件句柄泄漏
author: Victor
date: 2018-07-05 20:16:42
tags:
categories:
---
# 代码
```
public class FileHandleLeakExample {

  public static String readContentFromFile(String filename) {

    StringBuilder sb = new StringBuilder();
    BufferedReader br = null;
    try {
      br = new BufferedReader(new FileReader(filename));
      String line = null;
      while ((line = br.readLine()) != null) {
        sb.append(line).append("\n");
      }
    } catch (Exception e) {
      System.err.println(e.getMessage());
      exit(1);
    } finally {
//          模拟疏忽关掉句柄的操作
//            try {
//                br.close();
//            } catch (IOException e) {
//                e.printStackTrace();
//            }
    }
    return sb.toString();
  }

  public static void main(String[] args) {
    for (int j = 0; j < 20; j++) {
      System.out.println(j++);
      FileHandleLeakExample.readContentFromFile("/Users/victor/tmp/qingshu.txt");
    }
    System.out.println("byte");
  }

}
```

- 在System.out.println("byte"); 打断点
- 通过jps
- lsof -p pid 观察


![upload successful](/images/pasted-206.png)

# lsof命令
lsof（list open files）是一个列出当前系统打开文件的工具。在linux环境下，任何事物都以文件的形式存在，通过文件不仅仅可以访问常规数据，还可以访问网络连接和硬件。所以如传输控制协议 (TCP) 和用户数据报协议 (UDP) 套接字等，系统在后台都为该应用程序分配了一个文件描述符，无论这个文件的本质如何，该文件描述符为应用程序与基础操作系统之间的交互提供了通用接口。因为应用程序打开文件的描述符列表提供了大量关于这个应用程序本身的信息，因此通过lsof工具能够查看这个列表对系统监测以及排错将是很有帮助的。

## lsof打开的文件可以是：

1.普通文件
2.目录
3.网络文件系统的文件
4.字符或设备文件
5.(函数)共享库
6.管道，命名管道
7.符号链接
8.网络文件（例如：NFS file、网络socket，unix域名socket）
9.还有其它类型的文件，等等


## 参数
lsof abc.txt 显示开启文件abc.txt的进程
lsof -c abc 显示abc进程现在打开的文件
lsof -c -p 1234 列出进程号为1234的进程所打开的文件
lsof -g gid 显示归属gid的进程情况
lsof +d /usr/local/ 显示目录下被进程开启的文件
lsof +D /usr/local/ 同上，但是会搜索目录下的目录，时间较长
lsof -d 4 显示使用fd为4的进程
lsof -i 用以显示符合条件的进程情况
lsof -i[46] [protocol][@hostname|hostaddr][:service|port]
  46 --> IPv4 or IPv6
  protocol --> TCP or UDP
  hostname --> Internet host name
  hostaddr --> IPv4地址
  service --> /etc/service中的 service name (可以不止一个)
  port --> 端口号 (可以不止一个)

## 说明：

```
lsof输出各列信息的意义如下：

COMMAND：进程的名称

PID：进程标识符

PPID：父进程标识符（需要指定-R参数）

USER：进程所有者

PGID：进程所属组

FD：文件描述符，应用程序通过文件描述符识别该文件。如cwd、txt等

（1）cwd：表示current work dirctory，即：应用程序的当前工作目录，这是该应用程序启动的目录，除非它本身对这个目录进行更改
（2）txt ：该类型的文件是程序代码，如应用程序二进制文件本身或共享库，如上列表中显示的 /sbin/init 程序
（3）lnn：library references (AIX);
（4）er：FD information error (see NAME column);
（5）jld：jail directory (FreeBSD);
（6）ltx：shared library text (code and data);
（7）mxx ：hex memory-mapped type number xx.
（8）m86：DOS Merge mapped file;
（9）mem：memory-mapped file;
（10）mmap：memory-mapped device;
（11）pd：parent directory;
（12）rtd：root directory;
（13）tr：kernel trace file (OpenBSD);
（14）v86  VP/ix mapped file;
（15）0：表示标准输出
（16）1：表示标准输入
（17）2：表示标准错误

一般在标准输出、标准错误、标准输入后还跟着文件状态模式：r、w、u等

（1）u：表示该文件被打开并处于读取/写入模式
（2）r：表示该文件被打开并处于只读模式
（3）w：表示该文件被打开并处于
（4）空格：表示该文件的状态模式为unknow，且没有锁定
（5）-：表示该文件的状态模式为unknow，且被锁定

同时在文件状态模式后面，还跟着相关的锁

（1）N：for a Solaris NFS lock of unknown type;
（2）r：for read lock on part of the file;
（3）R：for a read lock on the entire file;
（4）w：for a write lock on part of the file;（文件的部分写锁）
（5）W：for a write lock on the entire file;（整个文件的写锁）
（6）u：for a read and write lock of any length;
（7）U：for a lock of unknown type;
（8）x：for an SCO OpenServer Xenix lock on part      of the file;
（9）X：for an SCO OpenServer Xenix lock on the      entire file;
（10）space：if there is no lock.

TYPE：文件类型，如DIR、REG等，常见的文件类型

（1）DIR：表示目录
（2）CHR：表示字符类型
（3）BLK：块设备类型
（4）UNIX： UNIX 域套接字
（5）FIFO：先进先出 (FIFO) 队列
（6）IPv4：网际协议 (IP) 套接字

DEVICE：指定磁盘的名称

SIZE：文件的大小

NODE：索引节点（文件在磁盘上的标识）

NAME：打开文件的确切名称

```

## 例子


### 查找谁在使用文件系统


在卸载文件系统时，如果该文件系统中有任何打开的文件，操作通常将会失败。那么通过lsof可以找出那些进程在使用当前要卸载的文件系统，如下：
 lsof /GTES11/ 
COMMAND PID USER FD TYPE DEVICE SIZE NODE NAME 
bash 4208 root cwd DIR 3,1 4096 2 /GTES11/ 
vim 4230 root cwd DIR 3,1 4096 2 /GTES11/
在这个示例中，用户root正在其/GTES11目录中进行一些操作。一个 bash是实例正在运行，并且它当前的目录为/GTES11，另一个则显示的是vim正在编辑/GTES11下的文件。要成功地卸载/GTES11，应该在通知用户以确保情况正常之后，中止这些进程。 这个示例说明了应用程序的当前工作目录非常重要，因为它仍保持着文件资源，并且可以防止文件系统被卸载。这就是为什么大部分守护进程（后台进程）将它们的目录更改为根目录、或服务特定的目录（如 sendmail 示例中的 /var/spool/mqueue）的原因，以避免该守护进程阻止卸载不相关的文件系统。

### 恢复删除的文件


当Linux计算机受到入侵时，常见的情况是日志文件被删除，以掩盖攻击者的踪迹。管理错误也可能导致意外删除重要的文件，比如在清理旧日志时，意外地删除了数据库的活动事务日志。有时可以通过lsof来恢复这些文件。
当进程打开了某个文件时，*只要该进程保持打开该文件，即使将其删除，它依然存在于磁盘中*。这意味着，进程并不知道文件已经被删除，它仍然可以向打开该文件时提供给它的文件描述符进行读取和写入。除了该进程之外，这个文件是不可见的，因为已经删除了其相应的目录索引节点。
在/proc 目录下，其中包含了反映内核和进程树的各种文件。/proc目录挂载的是在内存中所映射的一块区域，所以这些文件和目录并不存在于磁盘中，因此当我们对这些文件进行读取和写入时，实际上是在从内存中获取相关信息。大多数与 lsof 相关的信息都存储于以进程的 PID 命名的目录中，即 /proc/1234 中包含的是 PID 为 1234 的进程的信息。每个进程目录中存在着各种文件，它们可以使得应用程序简单地了解进程的内存空间、文件描述符列表、指向磁盘上的文件的符号链接和其他系统信息。lsof 程序使用该信息和其他关于内核内部状态的信息来产生其输出。所以lsof 可以显示进程的文件描述符和相关的文件名等信息。也就是我们通过访问进程的文件描述符可以找到该文件的相关信息。
当系统中的某个文件被意外地删除了，只要这个时候系统中还有进程正在访问该文件，那么我们就可以通过lsof从/proc目录下恢复该文件的内容。 假如由于误操作将/var/log/messages文件删除掉了，那么这时要将/var/log/messages文件恢复的方法如下：
首先使用lsof来查看当前是否有进程打开/var/logmessages文件，如下：
` lsof |grep /var/log/messages `
syslogd 1283 root 2w REG 3,3 5381017 1773647 /var/log/messages (deleted)
从上面的信息可以看到 PID 1283（syslogd）打开文件的文件描述符为 2。同时还可以看到/var/log/messages已经标记被删除了。因此我们可以在 /proc/1283/fd/2 （fd下的每个以数字命名的文件表示进程对应的文件描述符）中查看相应的信息，如下：
` head -n 10 /proc/1283/fd/2 `
Aug 4 13:50:15 holmes86 syslogd 1.4.1: restart. 
Aug 4 13:50:15 holmes86 kernel: klogd 1.4.1, log source = /proc/kmsg started. 
Aug 4 13:50:15 holmes86 kernel: Linux version 2.6.22.1-8 (root@everestbuilder.linux-ren.org) (gcc version 4.2.0) #1 SMP Wed Jul 18 11:18:32 EDT 2007 Aug 4 13:50:15 holmes86 kernel: BIOS-provided physical RAM map: Aug 4 13:50:15 holmes86 kernel: BIOS-e820: 0000000000000000 - 000000000009f000 (usable) Aug 4 13:50:15 holmes86 kernel: BIOS-e820: 000000000009f000 - 00000000000a0000 (reserved) Aug 4 13:50:15 holmes86 kernel: BIOS-e820: 0000000000100000 - 000000001f7d3800 (usable) Aug 4 13:50:15 holmes86 kernel: BIOS-e820: 000000001f7d3800 - 0000000020000000 (reserved) Aug 4 13:50:15 holmes86 kernel: BIOS-e820: 00000000e0000000 - 00000000f0007000 (reserved) Aug 4 13:50:15 holmes86 kernel: BIOS-e820: 00000000f0008000 - 00000000f000c000 (reserved)
从上面的信息可以看出，查看 /proc/8663/fd/15 就可以得到所要恢复的数据。如果可以通过文件描述符查看相应的数据，那么就可以使用 I/O 重定向将其复制到文件中，如:
`cat /proc/1283/fd/2 > /var/log/messages`
对于许多应用程序，尤其是日志文件和数据库，这种恢复删除文件的方法非常有用。


# lsof对多线程的影响

```
mcu 13810 server1 7176r REG 8,1 1984229 2001190 /home/box/storage/kvdb/001125.ldb
mcu 13810 server1 7177r REG 8,1 1984229 2001190 /home/box/storage/kvdb/001125.ldb
mcu 13810 server1 7178r REG 8,1 1984229 2001190 /home/box/storage/kvdb/001125.ldb
mcu 13810 server1 7179r REG 8,1 1984229 2001190 /home/box/storage/kvdb/001125.ldb
mcu 13810 server1 7180r REG 8,1 1984229 2001190 /home/box/storage/kvdb/001125.ldb
mcu 13810 server1 7181r REG 8,1 1984229 2001190 /home/box/storage/kvdb/001125.ldb
mcu 13810 server1 7182r REG 8,1 1984229 2001190 /home/box/storage/kvdb/001125.ldb
```
lsof列出的多个线程并不代表每个线程占用了一个句柄，而是代表这些线程对这个文件有访问权限，实际上只要这个进程打开了这个文件，lsof都会列出所有子线程。

https://github.com/Tencent/phxpaxos/issues/15
## so 通过lsof观察如何数量过多,并不一定是文件句柄泄漏,可能是多线程进行访问

# file-leak-detector(待测试)
官网:
http://file-leak-detector.kohsuke.org/


# 参考
http://www.cnblogs.com/peida/archive/2013/02/26/2932972.html
https://www.cnblogs.com/ggjucheng/archive/2012/01/08/2316599.html
https://blog.coding.net/blog/java-file-leaks