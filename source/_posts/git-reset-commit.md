---
title: git撤销已提交但未push的代码
date: 2017-09-16 20:18:40
cdn: "header-off"
header-img: "/img/github.png"
tags:
	- git
---
 1. 查找想要回退到的版本号：
 	+ 在IDEA中可以："Version Control -> Log -> 右键提交记录 ->  Copy Revision Number"获取到
 	+ 命令行通过命令：
 	``` shell
	$ git log
	commit 0edf1b84e1aa60b0571704fcd0e136504878b2ff
	Author: yellow_river <river_yellow@163.com>
	Date:   Thu Sep 21 11:13:01 2017 +0800
    modified configuration

	commit 2baa4aa01a0741a9b0cfe08626d8174ce01978c8
	Author: huanghe_MAC <river_yellow@163.com>
	Date:   Wed Sep 20 22:15:58 2017 +0800
    new configurations

	commit fb21cc83168e11428fb3e502c4b4a8fab8af0ac1
	Author: yellow_river <river_yellow@163.com>
	Date:   Wed Sep 20 17:38:16 2017 +0800
    new page
	```
	commit后的就是版本号。

2. 撤销分为两种，一种是将代码撤销至与撤销版本完全一致的代码：
	``` shell
	$ git reset --hard commit_id
	```
	另一种是保留已提交代码：
	``` shell
	$ git reset commit_id
	```