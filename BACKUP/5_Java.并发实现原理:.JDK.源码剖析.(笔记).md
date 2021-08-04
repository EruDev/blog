# [Java 并发实现原理: JDK 源码剖析 (笔记)](https://github.com/EruDev/blog/issues/5)

## ch01 多线程基础
![栗子](https://raw.githubusercontent.com/EruDev/md-picture/master/img/1627977889244.png)

> main() 函数退出后，该线程是否会强制退出？整个线程是否会强制退出？

不会。在t1.start() 前面加一行代码 t1.setDaemon(true)。当 main() 函数退出，线程 t1 会跟着退出，整个进程也会退出

线程被分为两类：**守护线程和非守护线程，默认都是非守护线程。** 在Java中有一个规定：当所有的非守护线程退出后，整个JVM进程就会退出。例如，垃圾回收线程就是守护线程，它们在后台默默工作，当开发者的所有前台线程（非守护线程）都退出之后，整个JVM进程就退出了。

**锁的对象是什么？**

```java
class A{
    public void synchronized f1() {...}
    public static void synchronized f2() {...}
}
```

等价于如下代码：
```java
class A {
    public void f1 () {
        synchronized (this) {...}
    }

    public void f1 () {
        synchronized (A.class) {...}
    }
}
```
对于非静态函数，锁其实是加在对象 a 上面的；对于静态函数，锁是加在 A.class 上的

**锁的本质是什么？**

从程序角度来看，锁其实就是一个 "对象"：
> 打个比方，线程就是一个个游客，资源就是一个待参观的房子。这个房子每次只允许进去一个人，而锁就是这个房子的门卫。

1. 锁对象内部有个标志位 (state 变量)，记录自己有没有被某个线程占用（也就是记录当前有没有游客已经进入了房子）。最简单的情况是这个state有0、1两个取值，0表示没有线程占用这个锁，1表示有某个线程占用了这个锁。
2. 如果这个对象被某个线程占用，它得记录这个线程的thread ID，知道自己是被哪个线程占用了（也就是记录现在是谁在房子里面）。
3. 这个对象还得维护一个thread id list，记录其他所有阻塞的、等待拿这个锁的线程（也就是记录所有在外边等待的游客）。在当前线程释放锁之后（也就是把state从1改回0），从这个thread idlist里面取一个线程唤醒。

**volatile：64 位写入的原子性、内存可见性和禁止重排序**
