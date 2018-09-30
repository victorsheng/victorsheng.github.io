---
title: maven-resources插件
tags:
  - maven
  - springboot
abbrlink: 3284306327
date: 2018-07-03 15:20:38
categories:
---
# 官网
https://maven.apache.org/plugins/maven-resources-plugin/plugin-info.html

# 概述
Resources Plugin将Resource元素指定的文件复制到输出目录。以下三个变体仅在指定或默认资源和输出目录元素的方式上有所不同。 Resources插件有三个目标：

## resources：resources
将主源代码的资源复制到主输出目录。
此目标通常自动执行，因为它默认绑定到流程资源生命周期阶段。它始终使用project.build.resources元素指定资源，默认情况下使用project.build.outputDirectory指定复制目标。

## resources：testResources
将测试源代码的资源复制到测试输出目录。
此目标通常自动执行，因为它默认绑定到process-test-resources生命周期阶段。它始终使用project.build.testResources元素指定资源，默认情况下使用project.build.testOutputDirectory指定复制目标。

## resources：copy-resources
将资源复制到输出目录。
此目标要求您配置要复制的资源，并指定outputDirectory。


## filtering
将变量注入到变量中:  ${...} 


# 具体配置

```

    <resources>
      <resource>
        <directory>src/main/resources</directory>
      </resource>
      <resource>
        <directory>src/main/filters/${env}</directory>
        <includes>
          <include>**/*.xml</include>
          <include>**/*.conf</include>
        </includes>
      </resource>
    </resources>


      <plugin>
        <artifactId>maven-resources-plugin</artifactId>
        <configuration>
          <encoding>UTF8</encoding>
        </configuration>
      </plugin>
```

