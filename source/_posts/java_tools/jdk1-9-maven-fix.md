title: jdk1.9-maven-fix
tags:
  - java
  - maven
  - JDK1.9
categories: 
    java
date: 2018-01-08 13:59:00
---
# 背景
在安装了多个jdk版本后,发现maven命令不好使了

# 现象
-  maven-compiler-plugin:3.1:compile (default-compile) @ report 出现异常
```
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ report ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 60 source files to /Users/victor/code/egenieProjects/report/target/classes
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by lombok.javac.apt.Processor to field com.sun.tools.javac.processing.JavacProcessingEnvironment.processorClassLoader
WARNING: Please consider reporting this to the maintainers of lombok.javac.apt.Processor
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 5.198 s
[INFO] Finished at: 2018-01-08T14:04:55+08:00
[INFO] Final Memory: 35M/115M
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.1:compile (default-compile) on project report: Fatal error compiling: java.lang.NoSuchFieldError: pid -> [Help 1]
[ERROR]
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR]
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoExecutionException
```

# 解决
- 1.9
```
victordeMacBook-Pro:report victor$ mvn -v
Apache Maven 3.5.2 (138edd61fd100ec658bfa2d307c43b76940a5d7d; 2017-10-18T15:58:13+08:00)
Maven home: /usr/local/Cellar/maven/3.5.2/libexec
Java version: 9.0.1, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk-9.0.1.jdk/Contents/Home
Default locale: zh_CN, platform encoding: UTF-8
OS name: "mac os x", version: "10.13.2", arch: "x86_64", family: "mac"
```

- 执行命令
```
echo -n "JAVA_HOME=`/usr/libexec/java_home -v 1.8`" > ~/.mavenrc
```
- 1.8
```
victordeMacBook-Pro:report victor$ mvn -v
Apache Maven 3.5.2 (138edd61fd100ec658bfa2d307c43b76940a5d7d; 2017-10-18T15:58:13+08:00)
Maven home: /usr/local/Cellar/maven/3.5.2/libexec
Java version: 1.8.0_151, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "mac os x", version: "10.13.2", arch: "x86_64", family: "mac"
```

# 思考
虽然此种方法解决了这个问题,但具体是什么原因导致的此问题,还需进一步研究

# 参考
http://geeekr.com/fix-maven-java-version-mac-osx/