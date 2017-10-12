---
title: jdk自带虚拟机性能监控工具
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
| jinfo(Configuration Info for Java) | 显示虚拟机配置信息（鸡肋）   | 
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
jstat是用于监视虚拟机各种运行状态信息的命令行工具。它可以显示本地或者远程虚拟机进程中的类装载、内存、垃圾收集。JIT编译等运行数据，在没有GUI图形界面、只提供纯文本控制台环境的服务器上，它将是运行期定位虚拟机性能问题的首选工具。
### 参数说明
| 选项 | 作用 |
|----|----|
| -class | 监视类装载、卸载数量、总空间以及类装载所耗费的时间 |
| -gc |监视Java堆状况，包括Eden区、两个survivor区、老年代、永久代等的容量、已用空间、GC时间合计等信息|
| -gccapacity |监视内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大、最小空间|
| -gcutil |监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比|
| -gccause |与-gcutil功能一样，但是会额外输出导致上一次GC产生的原因|
| -gcnew |监视新生代GC状况|
| -gcnewcapacity |监视内容与-gcnew基本相同，输出主要关注使用到的最大、最小空间|
| -gcold |监视老年代GC状况|
| -gcoldcapacity |监视内容与-gcold基本相同，输出主要关注使用到的最大、最小空间|
| -gcpermcapacity |输出永久代使用到的最大、最小空间|
| -compiler |输出JIT编译器编译过的方法、耗时等信息|
| -printcompilation |输出已经被JIT编译的方法|

### 示例
``` console
$ jstat -gccause 4576 3000 4
   S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC
 96.56   0.00  25.06   0.03   7.78      4    0.927     0    0.000    0.927 Allocation Failure   No GC
 96.56   0.00  69.56   0.03   8.54      4    0.927     0    0.000    0.927 Allocation Failure   No GC
  0.00  99.98  26.45   3.25   9.65      5    0.991     0    0.000    0.991 Allocation Failure   No GC
  0.00  99.98  26.97   3.25   9.65      5    0.991     0    0.000    0.991 Allocation Failure   No GC

```
### 列表项含义
| 名称 | 含义 |
|:-----:|------|
| S0 |Survivor space 0|
| S1 |Survivor space 1|
| E|Eden space|
| O |Old space|
| P |Perm space |
| YGC |Number of young generation garbage collection (GC) events.|
| YGCT |Young generation garbage collection time.|
| FGC |Number of full GC events.|
| FGCT |Full garbage collection time.|
| GCT |Total garbage collection time.|
| LGGC |Cause of last garbage collection|
| GCC |Cause of current garbage collection|
参考：http://docs.oracle.com/javase/9/tools/tools-and-command-reference.htm#JSWOR596

## jmap(Memory Map for Java)
jmap的作用并不仅仅是为了获取dump文件，它还可以查询finalize执行队列、 Java堆和永久代的详细信息，如空间使用率、 当前用的是哪种收集器等。
### 参数说明
| 选项 | 作用 |
|:------:|-----|
| -dump |生成Java堆转储快照。参数： live — When specified, dumps only the live objects; if not specified, then dumps all objects in the heap.   format=b — Dumps the Java heap,. in hprof binary format    file=filename — Dumps the heap to filename|
| -finalizerinfo |显示在F-Queue中等待Finalizer线程执行finalize方法的对象（LINUX/SOLARIS）|
| -heap |显示Java堆详细信息，如使用哪种回收器、参数配置、分代状况等（LINUX/SOLARIS）|
| -histo |显示堆中对象统计信息，包括类、实例数量、合计容量|
| -permstat |以Classloader为统计口径显示永久代内存状态（LINUX/SOLARIS）|
| -F |当虚拟机进程对-dump选项没有响应时，可使用这个选项强制生成dump快照（LINUX/SOLARIS）|

### 示例
``` console
$ jmap -dump:format=b,file=test 7736
Dumping heap to D:\dev-tools\Java\jdk1.7.0_79\bin\test ...
Heap dump file created
```
### 其他获取堆转储快照的方式
 如果不使用jmap命令，要想获取Java堆转储快照，还有一些比较“暴力”的手段：譬如-XX：+HeapDumpOnOutOfMemoryError参数，可以让虚拟机在OOM异常出现之后自动生成dump文件，通过-XX：+HeapDumpOnCtrlBreak参数则可以使用[Ctrl]+[Break]键让虚拟机生成dump文件，又或者在Linux系统下通过Kill-3命令发送进程退出信号“吓唬”一下虚拟机，也能拿到dump文件。
## jhat(JVM Heap Analysis Tool)
Sun JDK提供jhat（JVM Heap Analysis Tool）命令与jmap搭配使用，来分析jmap生成的堆转储快照。 jhat内置了一个微型的HTTP/HTML服务器，生成dump文件的分析结果后，可以在浏览器中查看。 不过在实际工作中，除非手上真的没有别的工具可用，否则一般都不会去直接使用jhat命令来分析dump文件，主要原因有二：一是一般不会在部署应用程序的服务器上直接分析dump文件，即使可以这样做，也会尽量将dump文件复制到其他机器（用于分析的机器一般也是服务器，由于加载dump快照文件需要比生成dump更大的内存，所以一般在64位JDK、 大内存的服务器上进行）上进行分析，因为分析工作是一个耗时而且消耗硬件资源的过程，既然都要在其他机器进行，就没有必要受到命令行工具的限制了；另一个原因是jhat的分析功能相对来说比较简陋，VisualVM，以及专业用于分析dump文件的Eclipse Memory Analyzer、 IBM HeapAnalyzer等工具，都能实现比jhat更强大更专业的分析功能。 

## jstack(Stack Trace for Java)
### 参数说明
| 选项 | 作用 |
|:----:|-----|
| -F |当正常输出的请求不被响应时，强制输出线程堆栈|
| -l |除堆栈外，显示关于锁的附加信息|
| -m |如果调用到本地方法的话，可以显示C/C++的堆栈|
### tip
在JDK 1.5后，java.lang.Thread类新增了一个getAllStackTraces()方法用于获取虚拟机中所有线程StackTraceElement对象。 使用这个方法可以通过简单的几行代码就完成jstack的大部分功能，在实际项目中不妨调用这个方法做个管理员页面，可以随时使用浏览器来查看线程堆栈.
#### 代码实现
``` jsp
<%@page import="java.util.Map" %>
<html>
<head>
    <title>服务器线程信息</title>
</head>
<body>
	<pre>
	<%
    for (Map.Entry<Thread, StackTraceElement[]> stackTrace : Thread.
            getAllStackTraces().entrySet()) {
        Thread thread = (Thread) stackTrace.getKey();
        StackTraceElement[] stack = (StackTraceElement[]) stackTrace.getValue();
        if (thread.equals(Thread.currentThread()) {
            continue;
        }
        out.print("\n线程:" + thread.getName() + "\n");
        for (StackTraceElement element : stack) {
            out.print("\t" + element + "\n");
        }
    }%>
</pre>
</body>
</html>
```