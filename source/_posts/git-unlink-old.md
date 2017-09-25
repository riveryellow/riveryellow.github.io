---
title: git pull或push时报错："unable to unlink old 'xxx/xxx/xx'：Invalid argument"
date: 2017-09-24 16:18:49
cdn: "header-off"
header-img: "../../16/git-reset-commit/img/github.png"
tags:
	- git
---
> 在本地使用git pull或push代码时，提交、或拉取代码失败，并出现报错：
> error: unable to unlink old 'xxx/xxx/xx.xx': Invalid argument
> 说起来这个问题也出现过好几次了orz

大多数遇到这种情况，基本都是要更新或提交的文件被系统线程占用导致的。例如：tomcat正在运行时，js文件被用于合并，因而被占用，此时只要把服务暂停，重新拉取或提交即可。