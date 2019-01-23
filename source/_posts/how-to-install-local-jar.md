---
title: how-to-install-local-jar
date: 2019-01-23 19:04:29
tags:
categories:
---

# 问题
此种方法无法将jar包打入到fat-jar中
```
    <dependency>
      <groupId>hpbc</groupId>
      <artifactId>api</artifactId>
      <version>1.0</version>
      <scope>system</scope>
      <systemPath>${project.basedir}/lib/jpbc-api-2.0.0.jar</systemPath>
    </dependency>
    <dependency>
      <groupId>hpbc</groupId>
      <artifactId>plaf</artifactId>
      <version>1.0</version>
      <scope>system</scope>
      <systemPath>${project.basedir}/lib/jpbc-plaf-2.0.0.jar</systemPath>
    </dependency>
```

# 步骤
安装到maven项目中的lib目录下:
```
mvn install:install-file -Dfile=./lib/jpbc-api-2.0.0.jar -DgroupId=jpbc -DartifactId=api -Dversion=2.0.0 -Dpackaging=jar -DlocalRepositoryPath=lib
mvn install:install-file -Dfile=./lib/jpbc-plaf-2.0.0.jar -DgroupId=jpbc -DartifactId=plaf -Dversion=2.0.0 -Dpackaging=jar -DlocalRepositoryPath=lib
```

然后pom添加
```
  <repositories>
    <repository>
      <id>my-local-repo</id>
      <url>file://${project.basedir}/lib</url>
    </repository>
  </repositories>
```


# 参考:
https://stackoverflow.com/questions/2229757/maven-add-a-dependency-to-a-jar-by-relative-path