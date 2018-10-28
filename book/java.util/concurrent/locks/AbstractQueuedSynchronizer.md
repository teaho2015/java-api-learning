# java.util.concurrent.locks.AbstractQueuedSynchronizer


## 简介

翻译自Java Doc：

提供给阻塞锁和基于FIFO等待序列的相关同步器实现的一个框架。此类设计给那些依靠一个int来表示状态的同步器。子类必须定义方法去改变状态，
比如，acquired和released。然，由于要确保同步，子类需使用 `getState`, `setState` and `compareAndSetState`方法获取或更新状态。

子类最好被定义为非公共(non-public)内部(internal)工具类，供其封闭类（enclosing类，不懂译得对否，即相对来说前者是嵌套类，Nested class）。








## references



