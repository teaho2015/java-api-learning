# jstack

jstack是java虚拟机自带的一种堆栈跟踪（stack trace）工具。

来自[官方说明][link: 1]：
>The jstack command prints Java stack traces of Java threads for a specified Java process, core file, or remote debug server. For each Java frame, the full class name, method name, byte code index (BCI), and line number, when available, are printed.

## 用法

~~~

>jstack -h
Usage:
    jstack [-l] <pid>
        (to connect to running process)
    jstack -F [-m] [-l] <pid>
        (to connect to a hung process)
    jstack [-m] [-l] <executable> <core>
        (to connect to a core file)
    jstack [-m] [-l] [server_id@]<remote server IP or hostname>
        (to connect to a remote debug server)

Options:
    -F  to force a thread dump. Use when jstack <pid> does not respond (process is hung)
    -m  to print both java and native frames (mixed mode)
    -l  long listing. Prints additional information about locks
    -h or -help to print this help message

~~~

`pid`使用[jps](jps.md)查询。

*-F* 当 `jstack [-l] pid`没有响应时，强制输出thread dump。

*-m* 输出java和native c/c++框架的所有线程信息。

*-l* Long listing. 打印关于锁的额外的信息。例如java.util.concurrent中的ownable synchronizers列表。

*-h* 和 *-help*，输出帮助信息。



## 输出解析

摘自[java troubleshooting guide | Example 2-35](http://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr019.html#BABJBCHG):
~~~
//省略……

"Finalizer" daemon prio=10 tid=0x000b2800 nid=0x5 in Object.wait() [0xf3f7f000..0xf3f7f9c0]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0xf4000b40> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:116)
        - locked <0xf4000b40> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:132)
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:159)

//省略……
~~~

首行有如下关于线程的信息([Java trouble shooting guide | 2.19.1 Thread Dump](http://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr019.html#BABBEFFC))：

1. 线程名字，`"Finalizer"`。

2. 是否守护线程，`daemon`。

3. 线程优先级，`prio`。

4. 线程id，线程在内存位置，`tid`。

5. 本地线程（native thread）id，`nid`。

6. 线程动作，`Object.wait()`，当前thread dump的当前线程的正在做的动作。

7. 地址区间，`[0xf3f7f000..0xf3f7f9c0]`，估算的线程的栈地址区域。

8. 线程状态， `java.lang.Thread.State: WAITING`， 线程共有多少状态请看[Thread States for a Thread Dump](http://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr034.html#BABJFBFI)
   或[java.lang.Thread.State](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.State.html)。

余下的是线程调用栈。

9. 调用位置，`at java.lang.Object.wait(Native Method)`。

10. 线程调用修饰，`waiting on <0xf4000b40>`。

### 线程名字

从名字可以简单区分下，

`"GC task thread#0 (ParallelGC)"`gcthread（ParallelGC）;`"C2 CompilerThread1"`compile thread等等。

从名字可以知道什么jvm线程，什么是用户的线程。比如（如果没自己起名字的话），`"pool-n-thread-n"`。

当然，我们可以通过`Thread(String name)`和（线程池的话）实现`ThreadFactory`定制一个自己的factory，能够自定义线程名字等信息。

btw，不建议使用`public Thread(ThreadGroup group, String name)`等方法，因为ThreadGroup是不安全的。具体可参考：[[5]Effective Java | 第73条](#references)。

### 守护线程

摘自JavaDoc `Thread.setDaemon(boolean)`方法。
>Marks this thread as either a {@linkplain #isDaemon daemon} thread
 or a user thread. The Java Virtual Machine exits when the only
 threads running are all daemon threads.

### 线程状态

| 状态 | 解释 |
| :---: | :---: |
| NEW | The thread has not yet started. |
| RUNNABLE | The thread is executing in the JVM. |
| BLOCKED | The thread is blocked waiting for a monitor lock. |
| WAITING | The thread is waiting indefinitely for another thread to perform a particular action. |
| TIMED_WAITING | The thread is waiting for another thread to perform an action for up to a specified waiting time. |
| TERMINATED | The thread has exited. |

注：[这里](../language/monitor.md)会解释什么是monitor。

### 线程动作

摘自[JavaDump analysis.pdf](JavaDump analysis.pdf)(注：google到的，如有版权问题不能挂请email我):
* runnable：状态一般为RUNNABLE。
* in Object.wait()：等待区等待，状态为WAITING或TIMED_WAITING。
* waiting for monitor entry：迚入区等待，状态为BLOCKED。
* waiting on condition：等待区等待、被park。
* sleeping：休眠的线程，调用了Thread.sleep()。

### 线程调用修饰

摘自[JavaDump analysis.pdf](JavaDump analysis.pdf)
* locked <对象地址> (a 类名) 使用synchronized申请对象锁成功，监视器的拥有者。
* waiting to lock <对象地址> (a 类名) 使用synchronized申请对象锁未成功，在迚入区等待。
* waiting on <对象地址> (a 类名) 使用synchronized申请对象锁成功后，释放锁幵在等待区等待。
* parking to wait for <对象地址> park是基本的线程阻塞原语，不通过监视器在对象上阻塞。
  随concurrent包会出现的新的机制，与synchronized体系不同。


## 具体分析

首先，是使用`jstack`（linux下可使用`kill`）命令生成thread dump。

然后，一般很难单纯从Thread Dump直接看出问题。我们需要从源代码、Thread Dump和系统相互的互为的进行分析、推导。

再者，一个个地排除，缩小问题，看究竟是发生在操作系统还是JVM还是第三方的中间件、库还是自己写的代码是否锁住了同一个地址等等。

如果是用户代码，在对照thread dump的同时，理清代码结构思路，自己画一下像以前《操作系统》学的，简单的线程竞争资源的图也是好的。
在有GUI的系统里的话，使用TDA等工具，还会有一些debug建议给你。
当然，还有最权威的[Java Platform, Standard Edition Tools Reference](http://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/toc.html)里面，罗列了一大堆问题的分析或解决方式。

如下附带一些网上找资料时找到的案例，可参阅一番，找找手感：
[javacodegeeks | JVM: How to analyze Thread Dump](https://www.javacodegeeks.com/2012/03/jvm-how-to-analyze-thread-dump.html)

[[6]Java performance | 4.5节](#references)

[各种 Java Thread State 第一分析法则](http://www.cnblogs.com/zhengyun_ustc/archive/2013/03/18/tda.html)

[三个实例演示 Java Thread Dump 日志分析](http://www.cnblogs.com/zhengyun_ustc/archive/2013/01/06/dumpanalysis.html)

[虚拟机stack全分析](http://go-on.iteye.com/blog/1673894)

[How to Analyze Java Thread Dumps using Visual VM](https://performancetuningengineering.wordpress.com/2014/10/27/how-to-analyze-java-thread-dumps-using-visual-vm/)

这是一个Oracle的Thread Dump分析ppt[threaddumptroubleshooting.pptx](threaddumptroubleshooting.pptx)。

这是一个非常全面的Dump分析ppt[JavaDump analysis.pdf](JavaDump analysis.pdf)。

[cnblog | JVM调优之jstack找出最耗cpu的线程并定位代码](http://www.cnblogs.com/chengJAVA/p/5821218.html)


## 监控

额，在发生问题前或发生问题时，预防和监控有时也是蛮实用的，这能让我们防止更大的问题发生并且会给我们在debug阶段提供很多有用的信息。

* [Java performance[6]](#references)中2.4节有提及，如何在不同系统中监控锁竞争。

* 在application中或app外，增加一些监控措施，比如app内监控线程状态并发送预警，写一些shell scripts去监控thread、定期抓取thread转储信息，或者使用一些第三方工具等等。



## references

[1][Java Platform, Standard Edition Tools Reference | jstack][link: 1]

[2][stack overflow | How to analyze a java thread dump?][link: 2]

[3][cnblog | java命令--jstack 工具](http://www.cnblogs.com/kongzhongqijing/articles/3630264.html)

[4][Java Platform, Standard Edition Troubleshooting Guide](http://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/toc.html)

[5]Joshua Blosh.Effective Java,第2版[M].中国:机械工业出版社，2009

[6]Charlie Hunt,Binu John.Java性能优化权威指南[M].中国:人民邮电出版社,2014

[7][How threads work: more details](http://www.javamex.com/tutorials/threads/how_threads_work.shtml)

[8][csdn |  jstack(查看线程)、jmap(查看内存)和jstat(性能分析)命令](http://blog.csdn.net/rodulf/article/details/53415860)


[link: wikipedia | 内存泄漏]:https://zh.wikipedia.org/zh-cn/%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F
[link: 1]: http://docs.oracle.com/javase/8/docs/technotes/tools/windows/jstack.html
[link: 2]: https://stackoverflow.com/questions/7599608/how-to-analyze-a-java-thread-dump


