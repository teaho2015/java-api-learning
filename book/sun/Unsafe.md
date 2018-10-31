# sun.misc.Unsafe

## 简介

> A collection of methods for performing low-level, unsafe operations.
> Although the class and all methods are public, use of this class is
> limited because only trusted code can obtain instances of it.

Unsafe类，类的名字说明它是一个充满危险操作的类。Java提供两种操作系统层面上的操作方式，一个是Native方法，另一个是Unsafe类。
不过尽管Unsafe类的所有方法都是public的，但只有受信任的代码才能使用该类。

## 用途与作用

Unsafe可以获取某个属性等在内存中的位置，可以分配/释放内存等等。

一些典型方法:
~~~
public native void putInt(Object o, long offset, int x);
public native void putAddress(long address, long x);
public final native boolean compareAndSwapObject(Object o, long offset, Object expected, Object x);
public native long allocateMemory(long paramLong);
public native long reallocateMemory(long paramLong1, long paramLong2);
public native void freeMemory(long paramLong);
~~~
allocate、free，学过c的应该懂的。

### CAS

可知的，在Jdk的基础库中，很多类使用Unsafe，如：AQS，AtomicInteger。其中很多是用Unsafe去做CAS(compare and swap)。
CAS是指：比较旧值是否是期望值，如果是则替换为新值的一个原子操作。CAS机制保证了对值的修改是线程安全的。




## 危险操作

![FBI WARNING](fbi.jpg)

尽管如Unsafe类所说，只有受信任的代码才能使用该类，但是，我们可以通过反射机制跳过安全监测。

[关于Unsafe类的一点研究](https://www.jianshu.com/p/9819eb48716a)说到了如何跳过检测使用Unsafe，亲测可运行。




## references

[1] [Unsafe.java](http://www.docjar.com/html/api/sun/misc/Unsafe.java.html)















