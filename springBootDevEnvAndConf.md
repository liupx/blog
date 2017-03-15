---
title: SpringBoot开发环境与配置
date: 2017-03-15 12:00:00
categories: "SpringBoot"
tags: [Java,SpringBoot]
---
### 开发环境与配置
#### 系统要求
1. JDK: 1.7及以上（本文使用 JDK1.7）
2. Spring Framework： 4.1.5及以上（本文使用 Spring 4.3.7）
3. 项目构建工具：Maven 或者 Gradle （本文使用 Maven）
4. 官方在线构建项目：[地址](http://start.spring.io/) 
	1. 本文使用的SpringBoot版本为1.5.2 [官方文档地址](http://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/html/)

#### Spring Boot命令行界面
1. Spring Boot提供了命令行界面（Command Line Interface, CLI），可以用来运行和测试Spring Boot应用
2. 安装： 
	1. 下载文件
		- [spring-boot-cli-1.5.2.RELEASE-bin.zip](http://repo.spring.io/release/org/springframework/boot/spring-boot-cli/1.5.2.RELEASE/spring-boot-cli-1.5.2.RELEASE-bin.zip)
		- [spring-boot-cli-1.5.2.RELEASE-bin.tar.gz](http://repo.spring.io/release/org/springframework/boot/spring-boot-cli/1.5.2.RELEASE/spring-boot-cli-1.5.2.RELEASE-bin.tar.gz)
	2. 解压后可用。考虑将安装地址（解压后 /bin 所在目录）添加到系统path变量（windows系统）中。
	3. cmd中 运行 `spring --version` 后可看到 `Spring CLI v1.5.2.RELEASE`即可。
	4. 更多信息可参考解压后文件根目录中 </font><font style="color:#A52A2A">INSTALL.TXT </font>
3. 简单使用
	1. 命令行工具可直接运行创建的groovy文件。
	2. 例如创建 </font><font style="color:#A52A2A">app.groovy 文件</font>，内容如下：
	``` java
	@RestController
	class ThisWillActuallyRun {
    @RequestMapping("/")
    String home() {
        "Hello World!"
    	}
	}
    ```
	3. 运行：`spring run app.groovy`后访问 [localhost:8080](http://localhost:8080) 显示 
	```java
		Hello World!
	```