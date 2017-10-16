---
title: Spring Boot CLI安装
cdn: header-off
date: 2017-10-16 16:17:43
header-img: "/img/spring.png"
tags:
	- Spring
	- Java
---

> The Spring Boot CLI is a command line tool that can be used if you want to quickly prototype with Spring. It allows you to run Groovy scripts, which means that you have a familiar Java-like syntax, without so much boilerplate code.

## Windows安装
### 下载压缩包
可以直接从spring官网下载：
+ [spring-boot-cli-2.0.0.BUILD-SNAPSHOT-bin.zip](https://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/2.0.0.BUILD-SNAPSHOT/spring-boot-cli-2.0.0.BUILD-SNAPSHOT-bin.zip)
+ [spring-boot-cli-2.0.0.BUILD-SNAPSHOT-bin.tar.gz](https://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/2.0.0.BUILD-SNAPSHOT/spring-boot-cli-2.0.0.BUILD-SNAPSHOT-bin.tar.gz)

### 环境配置
将文件解压到指定文件夹后，根据INSTALL.txt配置环境：
+ 在环境变量中，将java版本配置到1.8；
+ 新建环境变量SPRING_CLI_HOME，指向刚刚解压的文件夹（如：D:\dev-tools\plugin\spring-2.0.0.BUILD-SNAPSHOT），并将__%SPRING_CLI_HOME%\bin__添加到环境变量Path中。

### 验证
``` console
$ spring --version
Spring CLI v2.0.0.BUILD-SNAPSHOT
```

## MacOS安装
### 安装
使用Homebrew可以直接安装到/usr/local/bin中：
``` console
$ brew tap pivotal/tap
$ brew install springboot
```
安装好之后，命令spring会自动注册到shell中。
