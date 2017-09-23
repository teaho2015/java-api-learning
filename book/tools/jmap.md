# jmap


>Prints shared object memory maps or heap memory details for a process, core file, or remote debug server.
This command is experimental and unsupported.

为Java进程、core文件或远程debug服务器打印共享对象映射或堆内存细节。

## 用法

~~~

Usage:
    jmap [option] <pid>
        (to connect to running process)
    jmap [option] <executable <core>
        (to connect to a core file)
    jmap [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    <none>               to print same info as Solaris pmap
    -heap                to print java heap summary
    -histo[:live]        to print histogram of java object heap; if the "live"
                         suboption is specified, only count live objects
    -clstats             to print class loader statistics
    -finalizerinfo       to print information on objects awaiting finalization
    -dump:<dump-options> to dump java heap in hprof binary format
                         dump-options:
                           live         dump only live objects; if not specified,
                                        all objects in the heap are dumped.
                           format=b     binary format
                           file=<file>  dump heap to <file>
                         Example: jmap -dump:live,format=b,file=heap.bin <pid>
    -F                   force. Use with -dump:<dump-options> <pid> or -histo
                         to force a heap dump or histogram when <pid> does not
                         respond. The "live" suboption is not supported
                         in this mode.
    -h | -help           to print this help message
    -J<flag>             to pass <flag> directly to the runtime system

~~~

### 参数

* `<no option>`

将会打印目标虚拟机中加载的每个共享对象的起始地址、映射大小以及共享对象文件的路径全称。这与Solaris的pmap工具比较相似。

* `-heap`

打印一个堆的概要的信息。

举个例子大致解释下：
~~~

>jmap -heap 5232
Attaching to process ID 5232, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.101-b13 //版本

using parallel threads in the new generation.  //说明新生代使用parnew
using thread-local object allocation.
Concurrent Mark-Sweep GC    //说明老年代使用CMS垃圾收集器

Heap Configuration: //配置信息
   MinHeapFreeRatio         = 40                    //-XX:MinHeapFreeRatio， minimum allowed percentage of free heap space (0 to 100) after a GC event. 默认值是40
   MaxHeapFreeRatio         = 70                    //-XX:MaxHeapFreeRatio， the maximum allowed percentage of free heap space (0 to 100) after a GC event. 默认值是70
   MaxHeapSize              = 786432000 (750.0MB)   //-XX:MaxHeapSize=size, 最大heap size
   NewSize                  = 44695552 (42.625MB)   //-XX:NewSize=szie, 新生代大小
   MaxNewSize               = 262144000 (250.0MB)   //-XX:MaxNewSize=size，最大的新生代大小
   OldSize                  = 89522176 (85.375MB)   //老年代大小，没有对应的java启动参数
   NewRatio                 = 2                     //-XX:NewRatio=ratio, 新生代和老生代的大小比例
   SurvivorRatio            = 8                     //-XX:SurvivorRatio=ratio，eden区和其中一个survivor区的比例（注意，是与其中一个survivor区，因为两个survivor区必是等大小）
   MetaspaceSize            = 21807104 (20.796875MB)//-XX:MaxMetaspaceSize=size，metaspace大小
   CompressedClassSpaceSize = 1073741824 (1024.0MB) //-XX:CompressedClassSpaceSize，默认1G，这个参数主要是设置Klass Metaspace的大小，不过这个参数设置了也不一定起作用，前提是能开启压缩指针，假如-Xmx超过了32G，压缩指针是开启不来的。如果有Klass Metaspace，那这块内存是和Heap连着的。更详细的解读请参考[你假笨 | JVM源码分析之Metaspace解密](http://lovestblog.cn/blog/2016/10/29/metaspace/)
   MaxMetaspaceSize         = 17592186044415 MB     //-XX:MaxMetaspaceSize=size,the maximum amount of native memory that can be allocated for class metadata.默认是不限制大小的
   G1HeapRegionSize         = 0 (0.0MB)             //-XX:G1HeapRegionSize=size，如果使用g1垃圾收集器，那么这里显示的是设定的region大小，默认region是1m到32m。

Heap Usage: //Heap使用
New Generation (Eden + 1 Survivor Space):
   capacity = 83427328 (79.5625MB)
   used     = 39811536 (37.96723937988281MB)
   free     = 43615792 (41.59526062011719MB)
   47.72001807369403% used
Eden Space:
   capacity = 74186752 (70.75MB)
   used     = 35999512 (34.331809997558594MB)
   free     = 38187240 (36.418190002441406MB)
   48.52552649831603% used
From Space:
   capacity = 9240576 (8.8125MB)
   used     = 3812024 (3.6354293823242188MB)
   free     = 5428552 (5.177070617675781MB)
   41.25309937389184% used
To Space:
   capacity = 9240576 (8.8125MB)
   used     = 0 (0.0MB)
   free     = 9240576 (8.8125MB)
   0.0% used
concurrent mark-sweep generation: //老年代
   capacity = 185057280 (176.484375MB)
   used     = 109427528 (104.35822296142578MB)
   free     = 75629752 (72.12615203857422MB)
   59.13170668022355% used
~~~

* `-histo[:live]`

>Prints a histogram of the heap.For each Java class, the number of objects, memory size in bytes, and the fully qualified class names are printed.

打印一个条形图(histogram)，里面包含对应每个Java类的对象数、内存占用大小(byte)、类全称的数据。(不懂为何叫histogram，在我看来只是个破表格。。)。
当加上`:live`时，只打印存活对象。

* `-clstats`

输出各Classloader的数据统计。

* `-finalizerinfo`

打印等待终结的对象信息。

* `-dump:[live,]format=b,file=<filename>`

生成hprof格式的java heap dump文件。可使用`jhat`或一些profiler工具分析。

* `-F`

强制输出选项。当使用jmap -dump或jmap -histo选项没响应时可在原命令中添加`-F`再运行。注：此option不支持live子选项。

* `-J<flag>`

指定传递给运行jmap的java进程的JVM参数。


## references

[1][csdn | jps命令使用](http://blog.csdn.net/fwch1982/article/details/7947451)

[2][Java Platform, Standard Edition Tools Reference | jmap](http://docs.oracle.com/javase/8/docs/technotes/tools/unix/jmap.html)

[3][你假笨 | JVM源码分析之Metaspace解密](http://lovestblog.cn/blog/2016/10/29/metaspace/)




[link: 5]: https://stackoverflow.com/a/25088276/6874711

[oracle | memorymanagement-whitepaper]: http://www.oracle.com/technetwork/java/javase/memorymanagement-whitepaper-150215.pdf


