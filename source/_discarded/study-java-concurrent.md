title: dubbo-monitor学习
date: 2018-01-24 20:31:33
tags:
categories:
---

# concurrent包使用
![upload successful](/images/pasted-43.png)
java.util.concurrent.ConcurrentHashMap
java.util.concurrent.atomic.AtomicLong


# spring声明周期
```
public interface InitializingBean {
    void afterPropertiesSet() throws Exception;
}

public interface DisposableBean {
    void destroy() throws Exception;
}
```


# springmvc配置
``` xml

    <mvc:view-controller path="/" view-name="redirect:/index.htm" />
    <bean class="org.springframework.web.servlet.view.ContentNegotiatingViewResolver">
       <property name="contentNegotiationManager">
           <!--过滤当前应用能够支持的请求类型-->
           <bean id="contentNegotiationManager" class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
               <property name="favorPathExtension" value="false" />
               <property name="favorParameter" value="false" />
               <property name="mediaTypes" >
                   <value>
                       json=application/json
                   </value>
               </property>
           </bean>
       </property>
        <property name="viewResolvers">
            <list>
                <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                    <property name= "viewClass"   value ="org.springframework.web.servlet.view.JstlView" />
                    <property name="prefix" value="WEB-INF/jsp/"/>
                    <property name="suffix" value=".jsp"/>
                </bean>
            </list>
        </property>
        <property name="defaultViews">
            <list>
                <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView" />
            </list>
        </property>
    </bean>

    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
        <property name="messageConverters">
            <list>
                <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"/>
            </list>
        </property>
    </bean>
    <bean  class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>

```


