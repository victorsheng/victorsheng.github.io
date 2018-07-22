---
title: 初始gradle-编译spring源码缺货cglib的jar包
date: 2018-07-19 22:02:22
tags:
    - gradle
    - spring
    - 源码
categories:
---
之前只是听说过,或者在本地通过idea 安装过今天遇到了一个实际的问题,下载spring源码后一部分spring.cglib的始终找不到,最终在spring-core.gradle中发现了奥秘
```
task cglibRepackJar(type: Jar) { repackJar ->
	repackJar.baseName = "spring-cglib-repack"
	repackJar.version = cglibVersion

	doLast() {
		project.ant {
			taskdef name: "jarjar", classname: "com.tonicsystems.jarjar.JarJarTask",
					classpath: configurations.jarjar.asPath
			jarjar(destfile: repackJar.archivePath) {
				configurations.cglib.each { originalJar ->
					zipfileset(src: originalJar)
				}
				// Repackage net.sf.cglib => org.springframework.cglib
				rule(pattern: "net.sf.cglib.**", result: "org.springframework.cglib.@1")
				// As mentioned above, transform cglib's internal asm dependencies from
				// org.objectweb.asm => org.springframework.asm. Doing this counts on the
				// the fact that Spring and cglib depend on the same version of asm!
				rule(pattern: "org.objectweb.asm.**", result: "org.springframework.asm.@1")
			}
		}
	}
}
```
之前只知道gradle通过grovy脚本语言比之前的maven pom.xml的插件方式会灵活很多,却不知道会如此之方便,是时候该系统的学习下gradle了