
## [说说进程和线程的却别?](https://www.jianshu.com/p/153cb15a4c4c)

进程是程序的一次执行，是系统资源分配和调度的独立单位，它的作用是使程序能并发执行提高资源利用率和吞吐率。    
进程是资源分配和调度的基本单位，每个进程各自拥有独立的内存，使得各进程内存地址相互隔离;    
而线程是操作系统能够调度的最小单位,一个进程中可以有多个线程，线程间共享同一个进程的内存空间;   
线程只占用少数系统资源，如程序计数器、寄存器和私有的可配置的堆栈；   
进程包含线程使用的堆和栈，包含线程间共享的主内存。

## 知道 synchronized 的原理么？
