---
title: Java多版本JDK共存配置
date: 2017-03-22 15:00:00
categories: "Java"
tags: [Java,JDK]
---
### Java多版本JDK共存配置
> 以JDK 1.8 与 JDK 1.7 共存为例；

1. 官网下载JDK安装文件。
- 设置JDK要安装的路径。
- 在环境变量值中新建  `JAVA7_HOME`  和  `JAVA8_HOME`  ，值为各自安装地址。
- 切换JDK版本时，可以通过修改  `JAVA_HOME=%JAVA7_HOME%`  或  `JAVA_HOME=%JAVA8_HOME%`  来切换。
	 > 注意：<br>修改之后，在cmd窗口里输入  `SET PATH=C:\`  (*cmd中修改环境变量，只影响本次会话，不会影响系统环境变量。但可以触发环境变量更新。免去重启系统之烦恼*)<br>
	 > 由于环境变量中  `%System32%`  在  `%JAVA_HOME%\bin%`  和  `%JAVA_HOME%\jre\bin%` 之前，优先级更高。<br>
	 > 所以安装JRE的时候，系统会将JRE路径下/bin里的  `java.exe/javaw.exe/javaws.exe`  放置到  `%System32%`  中。以便于用户无需配置JAVA路径也可以运行Java程序。<br>
	 > 因此，应考虑将  `%JAVA_HOME%\bin%`  和  `%JAVA_HOME%\jre\bin%` 的位置放到  `%System32%`  之前。使其优先级更高。会强制使系统运行在JDK目录下的相应程序。
- 默认安装（不建议安装公共JRE,开发者不需要非开发环境的java.exe...，原因见上文。）