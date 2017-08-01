# jps


> The jps command lists the instrumented Java HotSpot VMs on the target system. The command is limited to reporting information on JVMs for which it has the access permissions.

jps是显示当前所有java进程pid的命令。


## 用法

~~~
usage: jps [-help]
       jps [-q] [-mlvV] [<hostid>]

Definitions:
    <hostid>:      <hostname>[:<port>]
~~~

*-q*只显示`pid`，不显示`class`名称,`jar`文件名和传递给`main`方法的参数
~~~
>jps -q
12948
10780
~~~

*-m*输出传递给`main`方法的参数，在嵌入式jvm上可能是`null`。一般例子如下：
~~~
>jps -m
13360 Launcher (省略)
6932 Jps -m
13512 JpsMain hello world
10780
~~~

*-l*输出应用程序main class的完整package名 或者 应用程序的jar文件完整路径名
~~~
>jps -l
13360 org.jetbrains.jps.cmdline.Launcher
13512 com.tea.tools.jps.JpsMain
10780
10876 sun.tools.jps.Jps
~~~

*-v*展示传递到JVM的参数。
~~~
>jps -v
13360 Launcher -Xmx700m -Djava.awt.headless=true （省略n个字）
13512 JpsMain -agentlib:jdwp=transport=dt_shmem,address=javadebug,suspend=y,server=n -Dfile.encoding=UTF-8
10780  -Xms128m -Xmx750m -XX:MaxPermSize=350m （省略n个字）
9996 Jps -Denv.class.path=D:\Program Files\Java\jdk1.8.0_101\lib\tools.jar;D:\Program Files\Java\jdk1.8.0_101\jre\lib\rt.jar; -Dapplication.home=D:\Program Files\Java\jdk1.8.0_101 -Xms8m
~~~

*-V*官方描述和我亲手操作结果是不同的。以下出自：[Java Platform, Standard Edition Tools Reference(windows) | jps | options](http://docs.oracle.com/javase/8/docs/technotes/tools/windows/jps.html#CHDIHFAD)
> Suppresses the output of the class name, JAR file name, and arguments passed to the main method, producing only a list of local JVM identifiers.

以上描述和*-q*的文档描述是相同的，可是，从我机器上的如下输出看出，`jps -V`展示了main class名和pid。

所以这里我是有存疑的。

~~~
\>jps -V
13360 Launcher
10836 Jps
13512 JpsMain
10780
~~~

*-J*`option` Passes option to the JVM.向JVM传递参数。

由前面的例子也可看出，jps是一个java程序，*-J*`option`就是在启动jps时，允许我们用户设置一些参数。
~~~
>jps -v -J-Xms612m
13252 Jps -Denv.class.path=D:\Program Files\Java\jdk1.8.0_101\lib\tools.jar;D:\Program Files\Java\jdk1.8.0_101\jre\lib\r
t.jar; -Dapplication.home=D:\Program Files\Java\jdk1.8.0_101 -Xms8m -Xms612m
//其余输出省略
~~~

## 原理

java程序在启动后，会在当前用户的环境变量中的`TMP`、`TEMP`(注意是System.getenv()中的)指定的目录下面，生成`\hsperfdata_<username>\<pid>`文件。

如下是我`<TMP>\hsperfdata_<username>`文件夹的文件列表：
~~~
>dir
 驱动器 C 中的卷没有标签。
 卷的序列号是 0008-7F34

 C:\Users\<username>\AppData\Local\Temp\hsperfdata_<username> 的目录

  17:32    <DIR>          .
  17:32    <DIR>          ..
  10:35            65,536 10780
  17:32            65,536 17412
  17:32            65,536 19252
               3 个文件        196,608 字节
               2 个目录  5,394,743,296 可用字节
~~~
jps
~~~
17412 Launcher
19252 JpsMain
10780
14412 Jps
~~~

可看到，有三个在运行的java程序。

jps通过解析这些文件并生成结果输出。

stack overflow在该问题下的回答[How does JPS tool get the name of the main class or jar it is executing][link: 5]，
有Jps如何解析这些文件的线索。我们可以通过了解那些PerfData文件的格式，甚至去获得更多的内容，还可以写出自己的工具。


## references

[1][csdn | jps命令使用](http://blog.csdn.net/fwch1982/article/details/7947451)

[2][hollischuang's blog | Java命令学习系列（一）——Jps](http://www.hollischuang.com/archives/105)

[3][Java Platform, Standard Edition Tools Reference | jps](http://docs.oracle.com/javase/8/docs/technotes/tools/windows/jps.html)

[4][Java Platform, Standard Edition Troubleshooting Guide | 2.15 The jps Utility](http://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr015.html)

[5][stack overflow | How does JPS tool get the name of the main class or jar it is executing][link: 5]

[link: 5]: https://stackoverflow.com/a/25088276/6874711





