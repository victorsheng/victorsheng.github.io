title: study_tomcat_source
date: 2018-01-19 07:34:39
tags:
categories:
---
# Container基本结构
```
<?xml version='1.0' encoding='utf-8'?>

<Server port="8005" shutdown="SHUTDOWN">
    ……
  <Service name="Catalina">
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
    <Engine name="Catalina" defaultHost="localhost">
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>

      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log." suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
      </Host>
    </Engine>
  </Service>
</Server>
```


![upload successful](/images/pasted-31.png)



![upload successful](/images/pasted-32.png)
- Engine 表示一个Servlet引擎，它可以包含一个或多个子容器，比如Host或者Context容器；
- Host 表示一台虚拟的主机，它可以包含一系列Context容器；
- Context 表示一个唯一的ServletContext，一个 Context 对应一个 Web 工程，它可以包含一个 或多个Wrapper容器；
- Wrapper 表示一个独立的Servlet定义，即Wrapper本质就是对Servlet进行了一层包装。


![upload successful](/images/pasted-33.png)

![upload successful](/images/pasted-34.png)