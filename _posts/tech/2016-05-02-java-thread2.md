---
title: java线程
tag: java
---

## {{ page.title }}

<br/>

### interrupt
关于`Thread.interrupt`方法的注释的原文如下.

~~~java
/**
     * Interrupts this thread.
     *
     * <p> Unless the current thread is interrupting itself, which is
     * always permitted, the {@link #checkAccess() checkAccess} method
     * of this thread is invoked, which may cause a {@link
     * SecurityException} to be thrown.
     *
     * <p> If this thread is blocked in an invocation of the {@link
     * Object#wait() wait()}, {@link Object#wait(long) wait(long)}, or {@link
     * Object#wait(long, int) wait(long, int)} methods of the {@link Object}
     * class, or of the {@link #join()}, {@link #join(long)}, {@link
     * #join(long, int)}, {@link #sleep(long)}, or {@link #sleep(long, int)},
     * methods of this class, then its interrupt status will be cleared and it
     * will receive an {@link InterruptedException}.
     *
     * <p> If this thread is blocked in an I/O operation upon an {@link
     * java.nio.channels.InterruptibleChannel </code>interruptible
     * channel<code>} then the channel will be closed, the thread's interrupt
     * status will be set, and the thread will receive a {@link
     * java.nio.channels.ClosedByInterruptException}.
     *
     * <p> If this thread is blocked in a {@link java.nio.channels.Selector}
     * then the thread's interrupt status will be set and it will return
     * immediately from the selection operation, possibly with a non-zero
     * value, just as if the selector's {@link
     * java.nio.channels.Selector#wakeup wakeup} method were invoked.
     *
     * <p> If none of the previous conditions hold then this thread's interrupt
     * status will be set. </p>
     *
     * <p> Interrupting a thread that is not alive need not have any effect.
     *
     * @throws  SecurityException
     *          if the current thread cannot modify this thread
     *
     * @revised 6.0
     * @spec JSR-51
     */
~~~

第一段话是一些权限检查(暂时不管),重点是第二段话,第二段话是说,如果线程A正在被
`Object#wait()`或者`Thread#sleep()`或者`Thread#join()`阻塞,这个时候
在其它地方调用了`A.interrup()`,则A线程会清除interrupted的标志,并且抛出
`InterruptedException`.\\
除了注释中所说的情况外,在其它情况下(此时线程A正在正常的执行代码)调用了
`A.interrup()`,此时仅仅是给A线程设置interrupted的标志,A线程仍然正常执行.\\
java中应该是没有办法中断一个线程的执行.
