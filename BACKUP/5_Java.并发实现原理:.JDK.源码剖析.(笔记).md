# [Java 并发实现原理: JDK 源码剖析 (笔记)](https://github.com/EruDev/blog/issues/5)

## ch01 多线程基础
![栗子](https://raw.githubusercontent.com/EruDev/md-picture/master/img/1627977889244.png)

> main() 函数退出后，该线程是否会强制退出？整个线程是否会强制退出？

不会。在t1.start() 前面加一行代码 t1.setDaemon(true)。当 main() 函数退出，线程 t1 会跟着退出，整个进程也会退出

线程被分为两类：**守护线程和非守护线程，默认都是非守护线程。** 在Java中有一个规定：当所有的非守护线程退出后，整个JVM进程就会退出。例如，垃圾回收线程就是守护线程，它们在后台默默工作，当开发者的所有前台线程（非守护线程）都退出之后，整个JVM进程就退出了。
