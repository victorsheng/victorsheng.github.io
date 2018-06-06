---
title: logback-study
date: 2018-04-27 17:49:43
tags:
categories:
---
# LogBack、Slf4j和Log4j之间的关系
Slf4j是The Simple Logging Facade for Java的简称，是一个简单日志门面抽象框架，它本身只提供了日志Facade API和一个简单的日志类实现，一般常配合Log4j，LogBack，java.util.logging使用。Slf4j作为应用层的Log接入时，程序可以根据实际应用场景动态调整底层的日志实现框架(Log4j/LogBack/JdkLog…)。

LogBack和Log4j都是开源日记工具库，LogBack是Log4j的改良版本，比Log4j拥有更多的特性，同时也带来很大性能提升。详细数据可参照下面地址：Reasons to prefer logback over log4j。

# 组件

## appender

<appender>是<configuration>的子节点，是负责写日志的组件。<appender>有两个必要属性name和class。name指定appender名称，class指定appender的全限定名。

### ConsoleAppender
把日志添加到控制台，有以下子节点：

<encoder>：对日志进行格式化。（具体参数稍后讲解 ）
<target>：字符串 System.out 或者 System.err ，默认 System.out .

### FileAppender
把日志添加到文件，有以下子节点：

<file>：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
<append>：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。
<encoder>：对记录事件进行格式化。（具体参数稍后讲解 ）
<prudent>：如果是 true，日志会被安全的写入文件，即使其他的FileAppender也在向此文件做写入操作，效率低，默认是 false。

### RollingFIleAppender
滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。有以下子节点：

<file>：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
<append>：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。
<encoder>：对记录事件进行格式化。（具体参数稍后讲解 ）
<rollingPolicy>:当发生滚动时，决定 RollingFileAppender 的行为，涉及文件移动和重命名。
<triggeringPolicy >: 告知 RollingFileAppender 何时激活滚动。
<prudent>：当为true时，不支持FixedWindowRollingPolicy。支持TimeBasedRollingPolicy，但是有两个限制，1不支持也不允许文件压缩，2不能设置file属性，必须留空。

### RollingPolicy

#### TimeBasedRollingPolicy： 
最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责触发滚动。有以下子节点：
<fileNamePattern>: 必要节点，包含文件名及“%d”转换符，%d”可以包含一个Java.text.SimpleDateFormat指定的时间格式，如：%d{yyyy-MM}。如果直接使用 %d，默认格式是 yyyy-MM-dd。RollingFileAppender 的file字节点可有可无，通过设置file，可以为活动文件和归档文件指定不同位置，当前日志总是记录到file指定的文件（活动文件），活动文件的名字不会改变；如果没设置file，活动文件的名字会根据fileNamePattern 的值，每隔一段时间改变一次。“/”或者“\”会被当做目录分隔符。

<maxHistory>: 可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每个月滚动，且<maxHistory>是6，则只保存最近6

#### FixedWindowRollingPolicy： 
根据固定窗口算法重命名文件的滚动策略。有以下子节点：
<minIndex>:窗口索引最小值。

<maxIndex>:窗口索引最大值，当用户指定的窗口过大时，会自动将窗口设置为12。

<fileNamePattern >: 必须包含“%i”例如，假设最小值和最大值分别为1和2，命名模式为 mylog%i.log,会产生归档文件mylog1.log和mylog2.log。还可以指定文件压缩选项，例如，mylog%i.log.gz 或者 没有log%i.log.zip


#### SizeBasedTriggeringPolicy： 
查看当前活动文件的大小，如果超过指定大小会告知RollingFileAppender 触发当前活动文件滚动。只有一个节点:
<maxFileSize>:这是活动文件的大小，默认值是10MB。

## logger

<logger>
用来设置某一个包或者具体的某一个类的日志打印级别、以及指定<appender>。<logger>仅有一个name属性，一个可选的level和一个可选的additivity属性。

name：用来指定受此logger约束的某一个包或者具体的某一个类。
level：用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，还有一个特殊值INHERITED或者同义词NULL，代表强制执行上级的级别。
如果未设置此属性，那么当前logger将会继承上级的级别。
additivity：是否向上级logger传递打印信息。默认是true。
<logger>可以包含零个或多个<appender-ref>元素，标识这个appender将会添加到这个logger。

## root

也是<logger>元素，但是它是根logger。只有一个level属性，应为已经被命名为”root”.