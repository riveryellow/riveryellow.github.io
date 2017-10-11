---
title: 虚拟机性能监控与故障处理工具
date: 2017-10-11 16:45:44
cdn: "header-off"
header-img: "/img/jvm.jpg"
tags:
	- JVM
	- Java
---

| 名称        | 主要作用          | 
|------------|-------------|
| jps(JVM Process Status Tool)| 显示指定系统内所有的HotSpot虚拟机进程 |
| jstat(JVM Statistics Monitoring Tool)     | 用于手机HotSpot虚拟机各方面的运行数据      |
| jinfo(Configuration Info for Java) | 显示虚拟机配置信息      | 
|jmap(Memory Map for Java)| 生成虚拟机的内存转出快照（heapdump文件） |
|jhat(JVM Heap Analysis Tool)| 用于分析heapdump文件，它会建立一个HTTP/HTML服务器，让用户可以在浏览器上查看分析结果 |
|jstack(Stack Trace for Java)| 显示虚拟机的线程快照 |

## jps(JVM Process Status Tool)

jps可以列出正在运行的虚拟机进程，并显示虚拟机执行主类（main函数所在的类）名称以及这些进程的本地虚拟机唯一ID（Local Virtual Machine Identifier,LVMID）。对于本地虚拟机进程来说，LVMID与操作系统的进程ID（Process Identifier,PID）是一致的，使用Windows的任务管理器或者UNIX的ps命令也可以查询到虚拟机进程的LVMID，但如果同时启动了多个虚拟机进程，无法根据进程名称定位时，那就只能依赖jps命令显示主类的功能才能区分。

### 参数说明
| 选项 | 作用 |
|:----:|----|
| -q | 只输出LVMID，省略主类的名称 |
| -m | 输出虚拟机进程启动时传递给主类main()函数的参数 |
| -l | 输出主类的全名，如果进程执行的是Jar包，输出Jar的路径 |
| -v | 输出虚拟机进程启动时JVM参数 |

### 示例
``` console
$ jps -l
6148 org.jetbrains.jps.cmdline.Launcher
9004 org.apache.catalina.startup.Bootstrap
9912 sun.tools.jps.Jps
5648
7916 org.jetbrains.idea.maven.server.RemoteMavenServer
```

## jstat(JVM Statistics Monitoring Tool)
``` console
$ jstat -gcutil 9004 3000 10
  S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT
  0.00  99.99  81.56   0.11   9.79      5    0.151     0    0.000    0.151
  0.00  99.99  81.56   0.11   9.79      5    0.151     0    0.000    0.151
  0.00  99.99  82.54   0.11   9.79      5    0.151     0    0.000    0.151
  0.00  99.99  89.74   0.11   9.79      5    0.151     0    0.000    0.151
  0.00  99.99  94.89   0.11   9.81      5    0.151     0    0.000    0.151
 99.93   0.00   2.29   2.86   9.82      6    0.251     0    0.000    0.251
 99.93   0.00   2.52   2.86   9.82      6    0.251     0    0.000    0.251
 99.93   0.00   3.35   2.86   9.82      6    0.251     0    0.000    0.251
 99.93   0.00   3.64   2.86   9.82      6    0.251     0    0.000    0.251
 99.93   0.00   4.08   2.86   9.82      6    0.251     0    0.000    0.251
```

## jinfo(Configuration Info for Java)

## jmap(Memory Map for Java)

## jhat(JVM Heap Analysis Tool)

## jstack(Stack Trace for Java)
