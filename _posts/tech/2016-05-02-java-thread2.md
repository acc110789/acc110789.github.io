---
title: java线程
tag: java
---

## {{ page.title }}

### interrupt
关于`Thread.interrupt`方法的注释的原文如下.

~~~java
    /**
     * <p> If this thread is blocked in an invocation of the {@link
     * Object#wait() wait()}, {@link Object#wait(long) wait(long)}, or {@link
     * Object#wait(long, int) wait(long, int)} methods of the {@link Object}
     * class, or of the {@link #join()}, {@link #join(long)}, {@link
     * #join(long, int)}, {@link #sleep(long)}, or {@link #sleep(long, int)},
     * methods of this class, then its interrupt status will be cleared and it
     * will receive an {@link InterruptedException}.
     */
~~~

如果线程A正在被`Object#wait()`或者`Thread#sleep()`或者`Thread#join()`阻塞,这个时候
在其它地方调用了`A.interrup()`,则A线程会清除interrupted的标志,并且抛出
`InterruptedException`.\\
除了注释中所说的情况外,在其它情况下(此时线程A正在正常的执行代码)调用了
`A.interrupt()`,此时仅仅是给A线程设置interrupted的标志,A线程仍然正常执行.\\
java中应该是没有办法正常中断(即不引起一些异常)一个线程的执行.

### State
线程也有生命周期,java将其生命周期划分为几种状态,通过调用`Thread.getState()`或者
直接将线程dump出来之后都能看到线程的状态.

~~~java
public enum State {
        /**
         * 即用new关键字创建了线程,但是还没有执行的状态.
         */
        NEW,

        /**
         * 线程正常执行代码时的状态,不是其它几种状态的状态.
         */
        RUNNABLE,

        /**
         * Thread state for a thread blocked waiting for a monitor lock.
         * A thread in the blocked state is waiting for a monitor lock
         * to enter a synchronized block/method or
         * reenter a synchronized block/method after calling
         * {@link Object#wait() Object.wait}.
         * 这个就解释的很好了,就是在等锁的状态.
         */
        BLOCKED,

        /**
         * Thread state for a waiting thread.
         * A thread is in the waiting state due to calling one of the
         * following methods:
         * <ul>
         *   <li>{@link Object#wait() Object.wait} with no timeout</li>
         *   <li>{@link #join() Thread.join} with no timeout</li>
         *   <li>{@link LockSupport#park() LockSupport.park}</li>
         * </ul>
         *
         * <p>A thread in the waiting state is waiting for another thread to
         * perform a particular action.
         *
         * For example, a thread that has called <tt>Object.wait()</tt>
         * on an object is waiting for another thread to call
         * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
         * that object. A thread that has called <tt>Thread.join()</tt>
         * is waiting for a specified thread to terminate.
         * 这种状态就是说本线程正在等待另一个线程完成某件事情后的通知.该状态和
         * BLOCKED状态的区别在于一个在等锁,一个在等另一个线程.
         */
        WAITING,

        /**
         * Thread state for a waiting thread with a specified waiting time.
         * A thread is in the timed waiting state due to calling one of
         * the following methods with a specified positive waiting time:
         * <ul>
         *   <li>{@link #sleep Thread.sleep}</li>
         *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
         *   <li>{@link #join(long) Thread.join} with timeout</li>
         *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
         *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
         * </ul>
         * 这个也是和WAITING状态大致相同,区别在于最多只等待指定的时间.
         */
        TIMED_WAITING,

        /**
         * Thread state for a terminated thread.
         * The thread has completed execution.
         * 就是线程的 Runnable.run()方法已经执行完了的状态.
         */
        TERMINATED;
    }
~~~

内容很好的一个链接[Thread state](http://journals.ecs.soton.ac.uk/java/tutorial/java/threads/states.html)

### alive
`Thread`有个方法是这样的.

~~~java
    /**
     * Tests if this thread is alive. A thread is alive if it has
     * been started and has not yet died.
     *
     * @return  <code>true</code> if this thread is alive;
     *          <code>false</code> otherwise.
     */
    public final native boolean isAlive();
~~~

感觉解释的还是挺模糊的,看了网上一些解释,有的说是start方法开始运行后,一直到run方法结束完毕
(抛出Exception也算结束)在这个期间算是alive.感觉这个说法不是很准确.我感觉这么定义:
当`getState`返回的不是`NEW`和`TERMINATED`是都算是alive,否则就不是alive.
也有人认为比如A,B两个线程,A调用了`B.start()`之后B就算是alive了,当B的run方法
执行完毕之后就不算是alive了.
[When is a Java thread alive?](http://stackoverflow.com/questions/17293304/when-is-a-java-thread-alive)
