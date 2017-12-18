---
title: mysql-5.7安装及初始化（Windows）
cdn: header-off
date: 2017-12-18 19:35:53
header-img: /img/mysql.jpg
tags:
	- mysql
---

由于mysql5.5与nodejs之间的兼容性问题，于是将win本上的mysql-5.5揪出来升级，这一折腾就是一下午。虽然不知道mysql-x.x.x会是什么样子，还会不会用同样安（ma）全（fan）的套路，还是做一下这一下午的踩坑记录吧。

## 下载、解压、环境变量
1. 照例地，下载依然选择DOWNLOADS -> COMMUNITY -> MYSQL COMMUNITY SERVER，社区版MYSQL zip，三百来M的样子。
2. 将下载好的压缩包解压到指定路径下，配置环境变量：
```
Path: %MYSQL_HOME%\bin
```
## 安装、初始化MYSQL
### 坑来了
+ win+R+cmd进入控制台，输入“net start mysql”启动mysql,出现报错：
``` console
	发生系统错误 2
	系统找不到指定的文件
```
+ 尝试进入mysql： mysql -u root -p （尝试了各种各样的密码），控制台不厌其烦地报着同样的错误：
``` console
ERROR 2003 (HY000): Can't connect to MySQL server on 'localhost' (10061)

```
跪了许久，决定投奔百度爸爸。
### 正确姿势
首先要解决的是第一个报错问题：
1. cmd进入到mysql安装目录下的bin目录下，执行mysqld --install，命令行提示（具体的忘了），服务已存在，路径是之前mysql5.5的安装路径；
2. 一不做二不休。mysqld --remove
``` console
Service successfully removed.
```
3. 重新安装： mysqld --install
``` console
Service successfully installed.
```
5. net start mysql，大功告成，准备吃鸡。
然而，初始密码依然无法命中，再次百度：
> mysql5.7新增的特性中主要的一方面就是极大增强了安全性，安装Mysql后默认会为root@localhost用户创建一个随机密码，这个随机密码在不同系统上需要使用不同方式查找，否则无法登录mysql并修改初始密码。

1. mysqld --initialize，进入官方指定mysql错误日志目录/data/，记事本打开一个以.err结尾的文件，可以看到：
	2017-12-18T11:30:37.626091Z 1 [Note] A temporary password is generated for root@localhost: >m:mdYE9XLga
其中“>m:mdYE9XLga”就是随机初始密码。
2. mysql -u root -p，输入随机密码，吃鸡！