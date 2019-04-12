---
title: Java线程池分析
cdn: header-off
date: 2018-06-11 16:10:20
header-img: "/img/java-banner.png"
tags:
    - Java
---

## 线程池创建参数

### int corePoolSize —— 线程池的基本线程数
> Core pool size is the minimum number of workers to keep alive (and not allow to time out etc) unless allowCoreThreadTimeOut is set, in which case the minimum is zero.

当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的 prestartAllCoreThreads() 方法，线程池会提前创建并启动所有基本线程。

### int maximumPoolSize —— 线程池最大线程数
> Maximum pool size. Note that the actual maximum is internally bounded by CAPACITY.

线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是如果使用了无界的任务队列这个参数就没什么效果。

### long keepAliveTime —— 闲置线程活动时间
> Timeout in nanoseconds for idle threads waiting for work. Threads use this timeout when there are more than corePoolSize present or if allowCoreThreadTimeOut. Otherwise they wait forever for new work.

线程池的工作线程空闲后保持存活的时间。如果任务很多，并且每个任务执行的时间比较短，可以调大这个参数，提高线程的利用率。

### TimeUnit unit —— 闲置线程活动时间单位
NANOSECONDS, MICROSECONDS, MILLISECONDS, SECONDS, MINUTES, HOURS, DAYS

### BlockingQueue<Runnable> workQueue —— 任务队列
> The queue used for holding tasks and handing off to worker threads.  We do not require that workQueue.poll() returning null necessarily means that workQueue.isEmpty(), so rely solely on isEmpty to see if the queue is empty (which we must do for example when deciding whether to transition from SHUTDOWN to TIDYING).  This accommodates special-purpose queues such as DelayQueues for which poll() is allowed to return null even if it may later return non-null when delays expire.

用于保存等待执行的任务的阻塞队列，例如：
+ ArrayBlockingQueue —— 一个基于数组结构的有界阻塞队列，此队列按 FIFO（先进先出）原则对元素进行排序。
+ LinkedBlockingDeque —— 一个基于链表结构的阻塞队列，此队列按 FIFO （先进先出） 排序元素，吞吐量通常要高于 ArrayBlockingQueue。静态工厂方法 Executors.newFixedThreadPool() 使用了这个队列。
+ SynchronousQueue —— 一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于 LinkedBlockingQueue，静态工厂方法 Executors.newCachedThreadPool() 使用了这个队列。

### ThreadFactory threadFactory
> Factory for new threads. All threads are created using this factory (via method addWorker).  All callers must be prepared for addWorker to fail, which may reflect a system or user's policy limiting the number of threads.  Even though it is not treated as an error, failure to create threads may result in new tasks being rejected or existing ones remaining stuck in the queue.
>
> We go further and preserve pool invariants even in the face of errors such as OutOfMemoryError, that might be thrown while trying to create the need to allocate a native stack in Thread.start, and users will want to perform clean pool shutdown to clean up.  There will likely be enough memory available for the cleanup code to complete without encountering yet another OutOfMemoryError.


### RejectedExecutionHandler handler —— 饱和策略
> Handler called when saturated or shutdown in execute.

当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是 AbortPolicy，表示无法处理新任务时抛出异常。以下是JDK1.8提供的四种策略：
+ AbortPolicy
> A handler for rejected tasks that throws a RejectedExecutionException.
直接抛出异常

+ CallerRunsPolicy
> A handler for rejected tasks that runs the rejected task directly in the calling thread of the execute method, unless the executor has been shut down, in which case the task is discarded.
只用调用者所在线程来运行任务。

+ DiscardPolicy
> A handler for rejected tasks that silently discards the rejected task.
不处理，丢弃掉。

+ DiscardOldestPolicy
> A handler for rejected tasks that discards the oldest unhandled request and then retries execute, unless the executor is shut down, in which case the task is discarded.
丢弃队列里最近的一个任务，并执行当前任务。

## 线程池工作流
