---
title: ContextLoaderListener详解
date: 2017-03-20 16:00:00
categories: "SpringWeb"
tags: [Java,SpringWeb]
---

## ContextLoaderListener详解
### 背景介绍 
Tomcat容器中的Java Web应用启动时，首先根据web.xml中配置的监听器ContextLoaderListener配置应用上下文(Application Context)。其中ContextLoaderListener究竟做了哪些工作；加载配置上下文时分为哪些阶段；在细节上又有哪些需要注意的地方？本文尝试从Spring源码的角度进行探究。
> 本文源码来自 `spring-web-4.3.4.RELEASE.jar` 。更多详细信息可参考spring文档 : [Spring 4.3.4RELEASE API:ContextLoaderListener](https://docs.spring.io/spring/docs/4.3.4.RELEASE/javadoc-api/org/springframework/web/context/ContextLoaderListener.html)
### ContextLoaderListener基本信息
#### 继承关系
 `ContextLoaderListener` 继承了 `ContextLoader` ，并且实现了 `ServletContextListener` 接口。
![ContextLoaderListener01.png](https://raw.githubusercontent.com/liupx/img/master/ContextLoaderListener01.png)
#### 成员方法
两个成员方法：
- contextInitialized() : 初始化Web应用上下文
- contextDestroyed() : 关闭Web应用上下文

### 上下文创建过程
#### 调用入口与配置  
ContextLoaderListener在 `web.xml` 文件的 `<listener>` 标签中创建。形如：
```xml
 <!-- Creates the Spring Container shared by all Servlets and Filters -->
 <listener>
      <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
 </listener>
```
<br>Web容器(e.g Tomcat)启动时扫描web.xml文件。如上文所示，由于继承了 `ServletContextListener` 接口，所以会在Web容器启动后，`ContextLoaderListener` 就收到事件初始化。 
<br>web.xml文件中 `<context-params>` 节点包含上下文类型(contextClass)和上下文配置路径(contextConfigLocation)两种配置参数。<br>
1. 其中，contextClass设置客户自定义的WebApplicationContext类型。
<br>当contextClass未找到时创建默认的 `XmlWebApplicationContext` 。
<br>源码出处如下：
```java
 protected Class<?> determineContextClass(ServletContext servletContext) {
        String contextClassName = servletContext.getInitParameter("contextClass");
        if(contextClassName != null) {
            try {
                return ClassUtils.forName(contextClassName, ClassUtils.getDefaultClassLoader());
            } catch (ClassNotFoundException var4) {
                throw new ApplicationContextException("Failed to load custom context class [" + contextClassName + "]", var4);
            }
        } else {
            contextClassName = defaultStrategies.getProperty(WebApplicationContext.class.getName());

            try {
                return ClassUtils.forName(contextClassName, ContextLoader.class.getClassLoader());
            } catch (ClassNotFoundException var5) {
                throw new ApplicationContextException("Failed to load default context class [" + contextClassName + "]", var5);
            }
        }
    }
```
2. <a id="contextConfigLocation">contextConfigLocation</a>用于设置applicatioContext*.xml文件的路径。不设置时默认为 `/WEB-INF/applicationContext.xml` 。多个上下文配置文件可用逗号或者空格分开。形如：

```xml
<!-- The definition of the Root Spring Container shared by all Servlets
        and Filters -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath:app-application-context.xml,
            classpath:app-datasource.xml
        </param-value>
    </context-param>
```
ContextLoaderListener的构造方法根据将根据 `<context-params>` 中的 `contextClass` 和 `contextConfigLocation` 这两个配置属性,调用初始化方法： `contextInitialized()` 创建Web应用程序上下文。
下文将首先从应用上下文的初始化过程进行探究。

#### initWebApplicationContext()
ContextLoaderListener继承ContextLoader类，其初始化过程 `contextInitialized()` 的实际工作由父类中 `initWebApplicationContext()` 完成。
<br>整体代码如下：
```java

    public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
        if(servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
            throw new IllegalStateException("Cannot initialize context because there is already a root application context present - check whether you have multiple ContextLoader* definitions in your web.xml!");
        } else {
            /*省略无关代码*/

            long startTime = System.currentTimeMillis();

            try {
                if(this.context == null) {
                    this.context = this.createWebApplicationContext(servletContext);
                }

                if(this.context instanceof ConfigurableWebApplicationContext) {
                    ConfigurableWebApplicationContext err = (ConfigurableWebApplicationContext)this.context;
                    if(!err.isActive()) {
                        if(err.getParent() == null) {
                            ApplicationContext elapsedTime = this.loadParentContext(servletContext);
                            err.setParent(elapsedTime);
                        }

                        this.configureAndRefreshWebApplicationContext(err, servletContext);
                    }
                }

                servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);
                ClassLoader err1 = Thread.currentThread().getContextClassLoader();
                if(err1 == ContextLoader.class.getClassLoader()) {
                    currentContext = this.context;
                } else if(err1 != null) {
                    currentContextPerThread.put(err1, this.context);
                }
                /*省略无关代码*/
                return this.context;
            } catch /*省略无关代码*/
        }
    }
```
##### createWebApplicationContext()
主要的逻辑在 `try{}` 部分。可知，当上下文为空时，调用 `createWebApplicationContext()` 创建Web应用上下文。
<br> 代码如下：
```java
protected WebApplicationContext createWebApplicationContext(ServletContext sc) {
        Class contextClass = this.determineContextClass(sc);
        if(!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
            throw new ApplicationContextException("Custom context class [" + contextClass.getName() + "] is not of type [" + ConfigurableWebApplicationContext.class.getName() + "]");
        } else {
            return (ConfigurableWebApplicationContext)BeanUtils.instantiateClass(contextClass);
        }
    }
```
- determineContextClass() 根据上文所说的 `contextClass` 设置上下文的类型。如前所述，未设置时，根据 `WebApplicationContext` 同路径下的 `ContextLoader.properties` 文件内容设置上下文类型。
<br>ContextLoader.properties 文件内容如下：
```xml
# Default WebApplicationContext implementation class for ContextLoader.
# Used as fallback when no explicit context implementation has been specified as context-param.
# Not meant to be customized by application developers.

org.springframework.web.context.WebApplicationContext=org.springframework.web.context.support.XmlWebApplicationContext

```
- 创建的上下文类型必须是 `ConfigurableWebApplicationContext` 的实例。
##### configureAndRefreshWebApplicationContext() 
创建完WebApplicationContext之后，就要调用 `configureAndRefreshWebApplicationContext()` 进行配置。
<br>通常是对刚刚创建的 `XmlWebApplicationContext` 进行初始化。
<br>代码如下：
```java
 protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac, ServletContext sc) {
        String configLocationParam;
        if(ObjectUtils.identityToString(wac).equals(wac.getId())) {
            configLocationParam = sc.getInitParameter("contextId");
            if(configLocationParam != null) {
                wac.setId(configLocationParam);
            } else {
                wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX + ObjectUtils.getDisplayString(sc.getContextPath()));
            }
        }

        wac.setServletContext(sc);
        configLocationParam = sc.getInitParameter("contextConfigLocation");
        if(configLocationParam != null) {
            wac.setConfigLocation(configLocationParam);
        }

        ConfigurableEnvironment env = wac.getEnvironment();
        if(env instanceof ConfigurableWebEnvironment) {
            ((ConfigurableWebEnvironment)env).initPropertySources(sc, (ServletConfig)null);
        }

        this.customizeContext(sc, wac);
        wac.refresh();
    }
``` 
首先是主要是设置一些基本信息：contextId,spring配置文件路径(contextConfigLocation,上文中提到的web.xml中配置[contextConfigLocation](#contextConfigLocation) --[test](#调用入口与配置))