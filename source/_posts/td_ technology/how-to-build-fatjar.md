---
title: 如何打fatjar包
date: 2018-11-08 18:10:38
tags:
categories:
---
# springboot插件
maven使用如下插件
```
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <executions>
          <execution>
            <goals>
              <goal>repackage</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
```

并且只有一个main方法,即可,无需META-INF下的配置

https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin/index.html

多main方法时,需要制定mainCLass
```
        <configuration>
          <mainClass>cn.victor123.demo.Connect</mainClass>
          <attach>false</attach>
        </configuration>
```

# 其他方法(待验证)
https://www.jianshu.com/p/8aa58813f80b


# 三种打包方法
## 非遮蔽方法（Unshaded）
非遮蔽是相对于遮蔽而说的，可以理解为一种朴素的办法。解压所有 jar 文件，再重新打包成一个新的单独的 jar 文件。

借助 Maven Assembly Plugin 都可以轻松实现非遮蔽方法的打包。

非遮蔽方法会把所有的 jar 包里的文件都解压到一个目录里，然后在打包同一个 fatjar 中。对于复杂应用很可能会碰到同名类相互覆盖问题。

## 遮蔽方法（Shaded）
遮蔽方法会把依赖包里的类路径进行修改到某个子路径下，这样可以一定程度上避免同名类相互覆盖的问题。最终发布的 jar 也不会带入传递依赖冲突问题给下游。

遮蔽方法依赖修改 class 的字节码，更新依赖文件的包路径达到规避同名同包类冲突的问题，但是改名也会带来其他问题，比如代码中使用 Class.forName 或 ClassLoader.loadClass 装载的类，Shade Plugin 是感知不到的。同名文件覆盖问题也没法杜绝，比如META-INF/services/javax.script.ScriptEngineFactory不属于类文件，但是被覆盖后会出现问题。


## 3. 嵌套方法（Jar of Jars）
还是一种办法就是在 jar 包里嵌套其他 jar，这个方法可以彻底避免解压同名覆盖的问题，但是这个方法不被 JVM 原生支持，因为 JDK 提供的 ClassLoader 仅支持装载嵌套 jar 包的 class 文件。所以这种方法需要自定义 ClassLoader 以支持嵌套 jar。


