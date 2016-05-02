---
title: 线程池,执行者
tag: java
---

## {{ page.title }}
今天把java中关于线程池的常用Api梳理一下

<br/>

### Executor

~~~ java
public interface Executor {
    /*
    说一点,该方法可能会抛出RejectedExecutionException,抛出的原因
    比如是因为当前的线程数量已经超出了线程池的maxPoolSize且相应的
    Queue也已经满了.
    */
    void execute(Runnable command);
}
~~~

<br/>

### ExecutorService
`ExecutorService`继承自`Executor`,多了生命周期这么一个概念

~~~ java
public interface ExecutorService extends Executor {
    /*
    不接受新提交的任务,但是已经提交的任务仍然会继续执行
    该方法不会等待之前提交的任务执行完毕(即该方法的执行是异步的)
    */
    void shutdown();

    /*
    尝试停止正在执行的任务,停止处理还在等待的任务并将这些正等待执行
    的任务作为一个List返回.
    该方法不会等待之前提交的任务执行完毕(即该方法的执行是异步的).
    该方法不能保证一定能是正在执行的任务停下来.通常停止正在执行的任务的
    手段是通过调用{@link Thread#interrupt}
    */
    List<Runnable> shutdownNow();

    /*
     Returns <tt>true</tt> if this executor has been shut down.
     一般来讲,就是说 {@link shutdown()} 或者 {@link shutdownNow()}在本方法
     执行之前执行过,如果执行过就返回true.
     */
    boolean isShutdown();

    /*
    就是说,在执行了 shutdownNow()或者shutdown()之后等所有活跃任务都已经停止
    了之后,这个时候执行笨方法就返回true
    */
    boolean isTerminated();

    /**
     * Blocks until all tasks have completed execution after a shutdown
     * request, or the timeout occurs, or the current thread is
     * interrupted, whichever happens first.
     * 这个我觉得英语解释的比我解释的清除.
     */
    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> Future<T> submit(Callable<T> task);

    <T> Future<T> submit(Runnable task, T result);

    Future<?> submit(Runnable task);
}
~~~

所以可以理解为ExecutorService有三个状态,一个就是正常状态,第二个是`shutdown`状态,
第三个是`terminated`状态.

<br/>

### ThreadPoolExecutor
`ThreadPoolExecutor`继承自`AbstractExecutorService`,是`ExecutorService`的具体
实现.\\
ThreadPoolExecutor有三个概念corePoolSize,maximumPoolSize,workQueue.\\
当一个任务通过execute(Runnable)方法欲添加到线程池时,采用如下的链条处理.

1. 如果此时线程池中的数量小于corePoolSize，即使线程池中的线程都处于空闲状态，
也要创建新的线程来处理被添加的任务。
2. 如果此时线程池中的数量大于等于corePoolSize，但是缓冲队列 workQueue未满，
那么任务被放入缓冲队列。
3. 如果此时线程池中的数量大于等于corePoolSize，缓冲队列workQueue满，
并且线程池中的数量小于maximumPoolSize，建新的线程来处理被添加的任务。
4. 如果此时线程池中的数量大于等于corePoolSize，缓冲队列workQueue满，并且线程池中的数量
等于maximumPoolSize，那么通过`RejectedExecutionHandler`所指定的策略来处理此任务。

也就是：
处理任务的优先级为：核心线程corePoolSize、任务队列workQueue、
最大线程maximumPoolSize，如果三者都满了，使用handler处理被拒绝的任务。

<br/>

核心和最大池的大小\\
ThreadPoolExecutor将根据corePoolSize(参见getCorePoolSize())
和maximumPoolSize(参见getMaximumPoolSize())设置的边界自动调整池大小.
当新任务在方法execute()中提交时,如果运行的线程少于
corePoolSize,则创建新线程来处理请求,即使其他线程是空闲的.如果运行的线程
多于corePoolSize而少于maximumPoolSize,则仅当队列满时才创建新线程.
如果设置的corePoolSize和maximumPoolSize相同,则创建了固定大小的线程池.
如果将maximumPoolSize设置为基本的无界值(如Integer.MAX_VALUE,
则允许池适应任意数量的并发任务.在大多数情况下,核心和最大池大小仅基于构造来设置,
不过也可以使用setCorePoolSize()和setMaximumPoolSize()进行动态更改。

<br/>
保持活动时间\\
如果池中当前有多于corePoolSize的线程,则这些多出的线程在空闲时间超过
keepAliveTime时将会终止(参见getKeepAliveTime()).
这提供了当池处于非活动状态时减少资源消耗的方法.如果池后来变得更为活动,则可以创建新的线程.
也可以使用方法setKeepAliveTime()动态地更改此参数.使用(Long.MAX_VALUE,TimeUnit.NANOSECONDS)
的值在关闭前有效地从以前的终止状态禁用空闲线程。

<br/>

注意的事情\\
因大于corePoolSize而小于等于maximumPoolSize额外申请的那部分线程在执行完其任务
之后如果还有新的任务的话,这个线程并不会立刻被回收,即使设置的keepAliveTime是0.而是
将新的任务安排给这个线程继续执行.线程是无序的,假设corePoolSize是1,maximumPoolSize
是2,这个pool产生的第一个线程是a,第二个线程是b,最后留下的线程不一定是a,有可能是b作为
corePoolSize而留下,即最后只保证有corePoolSize,不会保证具体是那个线程留下.
如果执行allowCoreThreadTimeOut(true)的话,corePoolSize里面的线程也会被回收.


