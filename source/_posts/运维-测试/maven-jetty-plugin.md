---
title: '通过mvn jetty:run启动项目'
abbrlink: 393526230
date: 2018-06-05 19:56:54
tags:
categories:
---
# maven配置例子

```
      <plugin>
        <groupId>org.eclipse.jetty</groupId>
        <artifactId>jetty-maven-plugin</artifactId>
        <version>9.2.8.v20150217</version>
        <configuration>
          <httpConnector>
            <port>8081</port>
          </httpConnector>
          <stopKey>shutdown</stopKey>
          <stopPort>9966</stopPort>
        </configuration>
      </plugin>
```

# 参考文档
http://www.blogjava.net/fancydeepin/archive/2012/06/23/maven-jetty-plugin.html

# 问题解决


```
[WARNING] Failed startup of context o.e.j.m.p.JettyWebAppContext@13192275{/,file:/Users/victor/code/tdProjects/td-opal-dataset/src/main/webapp/,STARTING}{file:/Users/victor/code/tdProjects/td-opal-dataset/src/main/webapp/}
org.eclipse.jetty.util.MultiException: Multiple exceptions
    at org.eclipse.jetty.annotations.AnnotationConfiguration.scanForAnnotations (AnnotationConfiguration.java:536)
    at org.eclipse.jetty.annotations.AnnotationConfiguration.configure (AnnotationConfiguration.java:447)
    at org.eclipse.jetty.webapp.WebAppContext.configure (WebAppContext.java:479)
    at org.eclipse.jetty.webapp.WebAppContext.startContext (WebAppContext.java:1337)
    at org.eclipse.jetty.server.handler.ContextHandler.doStart (ContextHandler.java:741)
    at org.eclipse.jetty.webapp.WebAppContext.doStart (WebAppContext.java:505)
    at org.eclipse.jetty.maven.plugin.JettyWebAppContext.doStart (JettyWebAppContext.java:365)
    at org.eclipse.jetty.util.component.AbstractLifeCycle.start (AbstractLifeCycle.java:68)
    at org.eclipse.jetty.util.component.ContainerLifeCycle.start (ContainerLifeCycle.java:132)
    at org.eclipse.jetty.util.component.ContainerLifeCycle.doStart (ContainerLifeCycle.java:114)
    at org.eclipse.jetty.server.handler.AbstractHandler.doStart (AbstractHandler.java:61)
    at org.eclipse.jetty.server.handler.ContextHandlerCollection.doStart (ContextHandlerCollection.java:163)
    at org.eclipse.jetty.util.component.AbstractLifeCycle.start (AbstractLifeCycle.java:68)
    at org.eclipse.jetty.util.component.ContainerLifeCycle.start (ContainerLifeCycle.java:132)
    at org.eclipse.jetty.util.component.ContainerLifeCycle.doStart (ContainerLifeCycle.java:114)
    at org.eclipse.jetty.server.handler.AbstractHandler.doStart (AbstractHandler.java:61)
    at org.eclipse.jetty.util.component.AbstractLifeCycle.start (AbstractLifeCycle.java:68)
    at org.eclipse.jetty.util.component.ContainerLifeCycle.start (ContainerLifeCycle.java:132)
    at org.eclipse.jetty.server.Server.start (Server.java:387)
    at org.eclipse.jetty.util.component.ContainerLifeCycle.doStart (ContainerLifeCycle.java:114)
    at org.eclipse.jetty.server.handler.AbstractHandler.doStart (AbstractHandler.java:61)
    at org.eclipse.jetty.server.Server.doStart (Server.java:354)
    at org.eclipse.jetty.maven.plugin.JettyServer.doStart (JettyServer.java:73)
    at org.eclipse.jetty.util.component.AbstractLifeCycle.start (AbstractLifeCycle.java:68)
    at org.eclipse.jetty.maven.plugin.AbstractJettyMojo.startJetty (AbstractJettyMojo.java:534)
    at org.eclipse.jetty.maven.plugin.AbstractJettyMojo.execute (AbstractJettyMojo.java:357)
    at org.eclipse.jetty.maven.plugin.JettyRunMojo.execute (JettyRunMojo.java:167)
    at org.apache.maven.plugin.DefaultBuildPluginManager.executeMojo (DefaultBuildPluginManager.java:137)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:208)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:154)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:146)
    at org.apache.maven.lifecycle.internal.LifecycleModuleBuilder.buildProject (LifecycleModuleBuilder.java:117)
    at org.apache.maven.lifecycle.internal.LifecycleModuleBuilder.buildProject (LifecycleModuleBuilder.java:81)
    at org.apache.maven.lifecycle.internal.builder.singlethreaded.SingleThreadedBuilder.build (SingleThreadedBuilder.java:56)
    at org.apache.maven.lifecycle.internal.LifecycleStarter.execute (LifecycleStarter.java:128)
    at org.apache.maven.DefaultMaven.doExecute (DefaultMaven.java:305)
    at org.apache.maven.DefaultMaven.doExecute (DefaultMaven.java:192)
    at org.apache.maven.DefaultMaven.execute (DefaultMaven.java:105)
    at org.apache.maven.cli.MavenCli.execute (MavenCli.java:956)
    at org.apache.maven.cli.MavenCli.doMain (MavenCli.java:290)
    at org.apache.maven.cli.MavenCli.main (MavenCli.java:194)
    at sun.reflect.NativeMethodAccessorImpl.invoke0 (Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke (NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke (DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke (Method.java:498)
    at org.codehaus.plexus.classworlds.launcher.Launcher.launchEnhanced (Launcher.java:289)
    at org.codehaus.plexus.classworlds.launcher.Launcher.launch (Launcher.java:229)
    at org.codehaus.plexus.classworlds.launcher.Launcher.mainWithExitCode (Launcher.java:415)
    at org.codehaus.plexus.classworlds.launcher.Launcher.main (Launcher.java:356)

```
https://stackoverflow.com/questions/50087942/gretty-build-fails-with-springboot