---
title: mavenjar包发布中遇到的问题
abbrlink: 3736081830
date: 2018-09-13 18:18:00
tags:
categories:
---

# Bamboo
Bamboo是澳大利亚公司Atlassian开发的产品，Atlassian公司以其产品项目跟踪软件JIRA和团队协同软件Confluence而闻名，Atlassian公司现有雇员500人，公司总部位于澳大利亚悉尼，在美国旧金山，荷兰阿姆斯特丹设有分公司。

Atlassian公司2002年由Mike Cannon-Brookes和Scott Farquhar创建，他们在澳大利亚新南威尔士大学认识，后辍学建立他们的企业软件公司。他们接受采访时说当时建立Atlassian是“直觉”上认为公司软件将会和互联网联系紧密。公司现有雇员500人，分布在悉尼、旧金山、阿姆斯特丹，2010年获得Accel Partner6000万美元风险投资。

而Atlassian Bamboo 是一款持续集成构建服务器软件(Build Server)（非开源软件）。 Bamboo的特点: 简单的用户界面 容易安装 - 顺利的话,5分钟内就可以让运行起来! 自动检测你的设置 - 如果您的Server上使用了Maven,Ant或者Java设置, Bamboo会自动检测他们; 连续的日志 - 监测你的build的colour coded日志; 容易显示所有项目。


# hubflow
http://datasift.github.io/gitflow/GitFlowForGitHub.html

# javadoc 语法

http://www.runoob.com/java/java-documentation.html




# maven-release-plugin插件
## 特性
- 把版本号后面的SNAPSHOT去掉(SNAPSHOT是可以被重复覆盖的), 向maven私服发布一个稳定版本;
- 需要在git上面建立tag, 作为release的milestone;
- 为下一个迭代进行SNAPSHOT版本的准备.

- 检查项目中所有的外部依赖包, 是否包含SNAPSHOT;
- 检查是否有未提交的代码;
- 运行单元测试, 确保全部通过.


## 使用步骤
- 在项目的pom文件中(如果是multi-module的只需要在主Pom中添加), 增加 和 , 无需添加mvn-release-plugin的依赖, 因为它默认被包含于maven的effective pom中;(项目级别)
- 检查自己的maven中的settings.xml是否包含了私服的用户名密码;(机器级别)
- 确保自己本地代码是在主分支, 并且是最新的副本, 否则后果自负;(现在是自动通过部署平台发布的保证了,pull最新的代码)
- 执行 mvn release:prepare, 这时插件会扫描项目依赖查看是否有SNAPSHOT, 是否存在未提交的文件, 确定当前release的版本号和下一个迭代的版本号, 插件会运行单元测试, 并向git中提交两次commit, 一次是release版本, 一次是下一个迭代的版本. 并将当前release版本打一个tag并提交到git上面去;(git操作)
- 执行 mvn release:perform, 插件会执行mvn deploy 操作, 并clean掉生成的缓存文件.(maven操作)


## maven发布地址设置
```
    <distributionManagement>
        <repository>
            <id>releases</id>
            <name>xxxx internal releases repository</name>
            <url>http://xxx.com/nexus/content/repositories/releases</url>
        </repository>
        <snapshotRepository>
            <id>snapshots</id>
            <name>xxxx internal snapshots repository</name>
            <url>http://xxx.com/nexus/content/repositories/snapshots</url>
        </snapshotRepository>
    </distributionManagement>
```
## git发布地址设置
```
    <scm>
        <url>http://xxx.com/project</url>
        <connection>- scm:git:git@xxx.com/project.git</connection>
        <developerConnection>- scm:git:git@xxx.com/project.git</developerConnection>
        <tag>HEAD</tag>
    </scm>
```

## 官网
http://maven.apache.org/maven-release/maven-release-plugin/

## 构建环节
- release:clean Clean up after a release preparation.
- release:prepare Prepare for a release in SCM.
- release:prepare-with-pom Prepare for a release in SCM, and generate release POMs that record the fully resolved projects used.
- release:rollback Rollback a previous release.
- release:perform Perform a release from SCM.
- release:stage Perform a release from SCM into a staging folder/repository.
- release:branch Create a branch of the current project with all versions updated.
- release:update-versions Update the versions in the POM(s).


# maven中snapshot快照库和release发布库的区别和作用
## 配置两个远端仓库
```
    <distributionManagement>
       <repository>
           <id>release</id>
           <name>demo Release</name>
           <url>https://maven.demo.com/nexus/content/repositories/releases</url>
       </repository>
       <snapshotRepository>
          <id>snapshot</id>
          <name>demo Snapshot</name>
          <url>https://maven.demo.com/nexus/content/repositories/snapshots</url>
       </snapshotRepository>
    </distributionManagement>
```
maven2会根据模块的版本号(pom文件中的version)中是否带有-SNAPSHOT来判断是快照版本还是正式版本。如果是快照版本，那么在mvn deploy时会自动发布到快照版本库中，而使用快照版本的模块，在不更改版本号的情况下，直接编译打包时，maven会自动从镜像服务器上下载最新的快照版本。如果是正式发布版本，那么在mvn deploy时会自动发布到正式版本库中，而使用正式版本的模块，在不更改版本号的情况下，编译打包时如果本地已经存在该版本的模块则不会主动去镜像服务器上下载。

所以，我们在开发阶段，可以将公用库的版本设置为快照版本，而被依赖组件则引用快照版本进行开发，在公用库的快照版本更新后，我们也不需要修改pom文件提示版本号来下载新的版本，直接mvn执行相关编译、打包命令即可重新下载最新的快照库了，从而也方便了我们进行开发。

应用snapshot和release库达到不同环境下发布不同的版本的目的

## 远端仓库认证(项目级别或者机器级别配置皆可以))
一般来说，分发构件到远程仓库需要认证，如果你没有配置任何认证信息，你往往会得到401错误。这个时候，如下在settings.xml中配置认证信息：

```
<settings>  
  ...  
  <servers>  
    <server>  
      <id>nexus-releases</id>  
      <username>admin</username>  
      <password>admin123</password>  
    </server>  
    <server>  
      <id>nexus-snapshots</id>  
      <username>admin</username>  
      <password>admin123</password>  
    </server>    
  </servers>  
  ...  
</settings>
```

# maven-scm-plugin插件
## 支持的api
The SCM Plugin has the following goals:

- scm:add - command to add file
- scm:bootstrap - command to checkout and build a project
- scm:branch - branch the project
- scm:changelog - command to show the source code revisions
- scm:check-local-modification - fail the build if there is any local modifications
- scm:checkin - command for commiting changes
- scm:checkout - command for getting the source code
- scm:diff - command for showing the difference of the working copy with the remote one
- scm:edit - command for starting edit on the working copy
- scm:export - command to get a fresh exported copy
- scm:list - command for get the list of project files
- scm:remove - command to mark a set of files for deletion
- scm:status - command for showing the scm status of the working copy
- scm:tag - command for tagging a certain revision
- scm:unedit - command to stop editing the working copy
- scm:update - command for updating the working copy with the latest changes
- scm:update-subprojects - command for updating all projects in a multi project build
- scm:validate - validate the scm information in the pom

## 底层支持的版本控制,如CVS,Git,Svn等

- Maven SCM AccuRev Provider	SCM Provider implementation for AccuRev (http://www.accurev.com/).
- Maven SCM Bazaar Provider	SCM Provider implementation for Bazaar (http://bazaar-vcs.org/).
- Maven SCM Clearcase Provider	SCM Provider implementation for Clearcase (http://www-306.ibm.com/software/awdtools/clearcase/).
- Maven SCM CVS Provider - Parent	SCM Provider implementation for CVS (http://www.nongnu.org/cvs/).
- Maven SCM Mercurial (Hg) Provider	SCM Provider implementation for Mercurial Hg (http://www.selenic.com/mercurial/wiki/).
- Maven SCM Git Provider - Parent	SCM Provider implementation for Git (http://git.or.cz/).
- Maven SCM Local Provider	SCM Provider implementation for Local.
- Maven SCM Perforce Provider	SCM Provider implementation for Perforce (http://www.perforce.com/).
- Maven SCM Standard Providers	Parent for all SCM providers supported.
- Maven SCM Starteam Provider	SCM Provider implementation for Starteam (http://www.borland.com/us/products/starteam/index.html).
- Maven SCM Subversion Provider - Parent	SCM Provider implementation for SVN (http://subversion.apache.org/).
- Maven SCM Synergy Provider	SCM Provider implementation for Synergy (http://www.telelogic.com/corp/products/synergy/index.cfm).
- Maven SCM Visual Source Safe Provider	SCM Provider implementation for VSS (http://msdn.microsoft.com/en-us/vstudio/aa700907.aspx).
- Maven SCM TFS Provider	A Maven 2 SCM Provider for Microsoft Visual Studio Team Foundation Server.
- Maven SCM MKS Integrity Provider	SCM Provider implementation for MKS Integrity : http://mks.com/
- Maven SCM Jazz/Rational Team Concert Provider	A Maven SCM Provider for IBM Jazz SCM (http://jazz.net/).

# 使用的maven插件列表
- maven-resources-plugin 复制resource目录至classPath
- maven-compiler-plugin 指定编译版本
- maven-jar-plugin
- maven-deploy-plugin
- maven-surefire-plugin
- maven-release-plugin
- 
