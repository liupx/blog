---
title: ContextLoaderListener详解
date: 2017-03-20 16:00:00
categories: "SpringWeb"
tags: [Java,SpringWeb]
---

### ContextLoaderListener详解
#### 背景介绍 
Tomcat容器中的Java Web应用启动时，首先根据web.xml中配置的监听器ContextLoaderListener配置应用上下文(Application Context)。其中ContextLoaderListener究竟做了哪些工作；加载配置上下文时分为哪些阶段；在细节上又有哪些需要注意的地方？本文尝试从Spring源码的角度进行探究。
> 本文源码来自`spring-web-4.3.4.RELEASE.jar`。更多详细信息可参考spring文档:[Spring 4.3.4RELEASE API:ContextLoaderListener](https://docs.spring.io/spring/docs/4.3.4.RELEASE/javadoc-api/org/springframework/web/context/ContextLoaderListener.html)

#### 创建、初始化与销毁
ContextLoaderListener在`web.xml`文件的`<listener>`标签中创建。形如：
```xml
 <!-- Creates the Spring Container shared by all Servlets and Filters -->
 <listener>
      <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
 </listener>
```
> Web容器(e.g Tomcat)启动时扫描web.xml文件，根据`<listener>`节点和随后的`<context-params>`节点找到上下文类型(contextClass)和上下文配置路径(contextConfigLocation)。随后的ContextLoaderListener的无参构造方法根据将根据这两个参数创建Web应用程序上下文。<br>其中，contextClass未找到时创建默认的XmlWebApplicationContext。<br>源码出处如下：

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
contextConfigLocation用于设置applicatioContext*.xml文件的路径。不设置时默认为/WEB-INF/applicationContext.xml。多个上下文配置文件可用逗号或者空格分开。形如：

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


两个成员方法：
- contextInitialized() : 初始化Web应用上下文
- contextDestroyed() : 关闭Web应用上下文

下文将首先从应用上下文的初始化过程进行探究。

#### 初始化
ContextLoaderListener继承ContextLoader类，其contextInitialized()实际工作由父类中initWebApplicationContext()完成。

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
其中，主要的逻辑在`try{}`部分。可知，当上下文为空时，创建Web应用上下文。创建的上下文类型必须是ConfigurableWebApplicationContext的实例。