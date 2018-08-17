线程对变量的修改都是在工作内存中进行的，那工作内存与主存之间的交互是如何进行的？

# 1. 8个原子操作

1. lock（锁定）： 作用于主内存变量，它把一个变量标识为一条线程独占的状态；
2.  unlock（解锁）：作用于主内存变量，它把一个被线程独占的变量释放出来，以便其他线程对其进行lock；
3. read（读取）：作用于主内存变量，它把一个变量的值从主存传输到工作内存中，以便随后的load操作使用；
4. load（载入）：作用于工作内存，它把read操作从主内存中得到的变量值放入工作内存中的变量副本中；
5. use（使用）：作用于工作内存，它把一个变量的值传递给执行引擎，每当虚拟机遇到需要使用变量的值的字节码指令时就会执行此操作；
6. assign（赋值）：作用于工作内存，它把从执行引擎接收到的值赋值给工作内存中的变量副本，每当虚拟机遇到一个为变量赋值的字节码指令时就会执行此操作；
7. store（存储）：作用于工作内存，它把工作内存中的一个变量的值传输到主内存，以便随后的write操作；
8. write（写入）：作用于主内存变量，把store操作从工作内存中得到的值放入主内存变量中。

![内存模型](https://upload-images.jianshu.io/upload_images/3631399-e7e16c1ca26f2481.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## volatile

- 可见性
- 禁止指令重排序

应用例子：DCL单例模式

# 3.原子性，可见性和有序性
1. 原子性：由Java内存模型直接保证的原子性操作包括read、load、use、assign、store、write，基本可以认为基本数据类型的读写都具有原子性，例外的有64位的long和double，但现在商用虚拟机这两个也是原子性操作。如果需要更大范围的原子性操作，Java内存模型还提供的lock和unlock来满足这种需求，尽管虚拟机并未把lock和unlock这两个指令开放给用户，但却提供了更高层次的字节码指令 monitorenter 和 monitorexit 来隐式实现这两个操作，反映到Java层就是 synchronized 关键字，所以 synchronized 语句块是具有原子性的。
2. 可见性：指的是一个线程修改了共享变量的值，其他线程能立即得知这个值的改变--volatile、final、Immutable 。
3. 有序性：线程内部表现为串行的有序操作，但从其他线程的角度看就是无序的--volatile 和 synchronized来保证有序性，volatile 是通过禁止指令重排序，而synchronized则是由“一个变量同一时间只能有一个线程对其进行lock操作”来获得。

# 4. 线程的实现
1. 使用内核线程实现

![LWP](https://upload-images.jianshu.io/upload_images/3631399-dcd9b8f79d7ff596.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

用户进程通过使用内核提供的轻量级进程（Light Weight Process，LWP）与内核线程一一对应。
- 由于基于内核线程实现的，所以对线程的操作都需要系统调用，在用户态和内核态之间切换成本较高；
- 轻量级进程需要消耗一定的内核资源，所以轻量级进程的数量是有限的。

2. 使用用户线程实现
![UT](https://upload-images.jianshu.io/upload_images/3631399-1c63f94e024726d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这种线程完全建立在用户空间上，对线程的建立、同步、销毁和调度都在用户空间完成，不需要内核的帮助，用户进程与线程是1:N的关系。
- 优点：不需要切换用户态和内核态，速度更快，也可以拥有更大数量的线程数量。
- 缺点：由于系统只将CPU资源分配到进程，诸如“阻塞如何处理”、“多核处理器如何将线程映射到其他处理器上”这类问题解决起来异常困难，甚至是不可能完成的。

> note: 现在使用这种方式的很少了。

3. 用户线程加轻量级进程混合实现
![lPW-UT](https://upload-images.jianshu.io/upload_images/3631399-1459479e4a3b2913.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

用户线程仍然完全建立在用户空间，使得用户线程的创建、调度、桥构操作变得较轻，同时并发支持的用户线程数量更多。操作系统提供的轻量级进程作为用户线程和内核线程的桥梁，这样可以使用内核提供的线程调度功能以及处理器映射，并且用户线程进行系统调用需要通过轻量级进程来完成，降低了整个进程阻塞的风险。用户线程与轻量级进程的是 N:M 的多对多关系。

# 5.Java线程的实现

JDK1.2以前，是基于“绿色线程”的用户线程实现的，而在JDK1.2中，线程模型替换为基于操作系统的原生线程模型来实现。因此，在目前的JDK版本中，操作系统支持怎么的线程模型，很大程度上决定了虚拟机的线程是怎样映射的，这点在不同的平台没有办法达成一致。对于Sun JDK来说，对于Windows和Linux平台是采用一对一的线程模型实现的，一条用户线程映射到一条轻量级进程中。

# 6.Java线程调度

协同式：线程自己决定什么时候让出CUP资源，早期的Windows，风险很大，容易阻塞。
抢占式：每个线程的执行时间由系统分配，线程状态的切换不由线程自身决定（可以自己放弃，Thread.yield()）。

线程优先级：Java线程对应10个优先级，同样处于Runable状态的两个线程，优先级高的线程可能会得到更多的执行时间，但这不是绝对的，所以不要依赖线程优先级来处理执行先后问题。一方面原因是，具体的操作系统可能并没有设置10个优先级（例如只有6个），这就导致在Java中设置为不同的优先级，映射到操作系统中是同一个优先级；另外一些原因，比如Windows下的优先级推进器。

# 7.线程的状态转换

![线程状态切换](https://upload-images.jianshu.io/upload_images/3631399-f94c2ab37ca19733.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 新建态（New）：新建但尚未启动的线程；
2. 运行（Runable）：正在运行或处于就绪态（Ready）;
3. 无限等待（waitting）：处于这种状态的线程不会分配CPU时间，需要等待其他线程对其唤醒，以下方法会让线程进入这个状态:
- 没有设置TimeOut参数的Object.wait();
- 没有设置TimeOut参数的Thread.join();
- LockSupport.park();

4. 期限等待（Timed waitting）: 处于这种状态的线程不会分配CPU时间，不过无需等待其他线程的唤醒，它会在等待一段时间后有系统自动唤醒，以下方法会导致线程进入该状态；
- Thread.sleep();
- 设置了TimeOut参数的Object.wait();
- 设置了TimeOut参数的Thread.join();
- LockSupport.parkNanos();
- LockSupport.parkUntil();

5. 阻塞（Blocking）：线程被阻塞了，与“等待状态”的区别是，“阻塞状态”在等待获取一个排它锁，这个事件由另外一个线程放弃这个锁触发，在程序等待进入同步代码块时，进入这种状态。

6. 结束（terminated）：已经终止的线程状态。