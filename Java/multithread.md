
### 1.理解synchronized的含义、明确synchronized关键字修饰普通方法、静态方法和代码块时锁对象的差异。

有如下一个类A
```
class A {
    public synchronized void a() {
    }
    public synchronized void b() {
    }
}
```
然后创建两个对象
```
A a1 = new A();
A a2 = new A();
```
然后在两个线程中并发访问如下代码：
```
Thread1                       Thread2
a1.a();                           a2.a();
```
请问二者能否构成线程同步？

如果A的定义是下面这种呢？
```
class A {
    public static synchronized void a() {
    }
    public static synchronized void b() {
    }
}
```
答：能不能实现线程同步，主要看它们是否在竞争同一个同步监视器，Java中的任何对象都可以作为同步监视器（锁对象）

- 对于实例同步方法，隐式锁是当前实例对象；

- 对于静态同步方法，隐式锁是类所对应的java.lang.Class对象；

- 对于同步代码块，锁是synchonized(obj)括号里配置的对象。

```
Thread1             Thread2          
a1.a();             a2.a();
 
```
如果a()是普通的同步方法，Thread1的同步监视器是a1，而Thread2的同步监视器是a2，监视器不同，无竞争可言，所以不构成线程同步，下面这种方式可构成线程同步。

```
Thread1             Thread2          
a1.a();             a1.a();
```

而如果将方法定义为静态同步方法，Thread1、Thread2的同步监视都是类A所对应的Class对象，在同一个虚拟机中，同一个类的Class对象是唯一的，所以构成线程同步。

上面这两种说法还存在一个漏洞，没有说是不是在同一个进程中。在Android中每个进程对应着一个不同的虚拟机，每个虚拟机的内存地址空间都是独立的，如果Thread1、Thread2属于不同的进程，类A所对应的Class对象也是两个不同的实例。

这个问题可以延伸出出一个更底层一点的问题：synchronized是如何实现一个线程获得同步锁后，其他的线程就不能获得的，或者是说对象锁的状态信息存在哪里？

对象锁的状态信息保存在对象的头信息中的Mark Word中，相关的还有Lock Record，当被锁定、解锁等时候这些状态信息就会改变，当然了，这些状态信息的改变也要保证原子性，否则也是不安全的。详情可移步《深入理解Java虚拟机：JVM高级特性与最佳实践》第五部分。

### 2.volatile关键字解决了什么问题
答：此关键字主要解决可见性和指令重重排序问题。不同的线程的工作区会存储主存变量的副本，有可能导致修改其他线程不可见的现象；对于指令重排序，有的编译器会将一些指令进行重排序，以提高性能，这有可能会引起一些问题，例如在DCL单例模式中，如果不加volatile修饰静态单例，就有可能得到一个不完全初始化的instance.

### 3.lock与unluck?



### 4.写一个死锁的例子

产生死锁的原因是两个或两个以上的线程都在等待对方持有的资源（同步锁对象），导致都处于僵持状态。下面这个例子，线程1持有lock1对象锁，线程2持有lock2对象锁，这个时候线程1在等待lock2，而线程2等待lock1，导致两个线程都处于阻塞状态。
```
public class DeadLock implements Runnable {
    static Object lock1 = new Object();
    static Object lock2 = new Object();

    boolean flag;

    public DeadLock(boolean flag) {
        this.flag = flag;
    }

    @Override
    public void run() {
        if (flag) {
            synchronized (lock1) {
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (lock2) {
                    System.out.println("lock2");
                }
            }
        } else {
            synchronized (lock2) {
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (lock1) {
                    System.out.println("lock1");
                }
            }
        }
    }
}
```

```
public class DeadLockDemo {
    public static void main(String...args) {
        DeadLock deadLock1 = new DeadLock(false);
        DeadLock deadLock2 = new DeadLock(true);
        new Thread(deadLock1).start();
        new Thread(deadLock2).start();
    }
}
```













