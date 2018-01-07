#1、Java中有哪几种引用？它们的含义和区别是什么？

从JDK1.2开始，Java中的引用类型分为四种，分别是：①强引用（StrongReference）、②软引用（softRefernce）、③弱引用（WeakReference）、④虚引用（PhantomReference）。

###强引用-StrongReference
这种引用是平时开发中最常用的，例如```String strong = new String("Strong Reference")```，当一个实例对象具有强引用时，垃圾回收器不会回收该对象，当内存不足时，宁愿抛出OutOfMemeryError异常也不会通过回收强引用的对象，因为JVM认为强引用的对象是用户正在使用的对象，它无法分辨出到底该回收哪个，强行回收有可能导致系统严重错误。

###软引用-SoftReference
如果一个对象只有软引用，那么只有当内存不足时，JVM才会去回收该对象，其他情况不会回收。软引用可以结合ReferenceQueue来使用，当由于系统内存不足，导致软引用的对象被回收了，JVM会把这个软引用加入到与之相关联的ReferenceQueue中。

```
ReferenceQueue referenceQueue = new ReferenceQueue();
SoftReference<Book> softReference = new SoftReference<>(new Book(), referenceQueue);
Book book = softReference.get();
Reference reference = referenceQueue.poll();
```
当系统内存不足时，触发gc，这个Book就会被回收，reference 将不为null。

###弱引用-WeakReference
只有弱引用的对象，当JVM触发gc时，就会回收该对象。与软引用不同的是，不管是否内存不足，弱引用都会被回收。弱引用可以结合ReferenceQueue来使用，当由于系统触发gc，导致软引用的对象被回收了，JVM会把这个弱引用加入到与之相关联的ReferenceQueue中，不过由于垃圾收集器线程的优先级很低，所以弱引用不一定会被很快回收。下面通过一个主动触发gc的例子来验证此结论。

```
 ReferenceQueue referenceQueue = new ReferenceQueue();
 WeakReference<Book> weakReference = new WeakReference(new Book(), referenceQueue);
 Book book = softReference.get();
 System.gc();
 //Runtime.getRuntime().gc();
 Reference reference = referenceQueue.poll();
```
![](/Resource/weak_reference.png)

当然这不是每次都能复现，因为我们调用System.gc()只是告诉JVM该回收垃圾了，但是它上面时候做还是不一定的，但就我测试来看，只要多写几次System.gc()，复现的概率还是很高的。

###虚引用-PhantomReference

如果一个对象只有虚引用在引用它，垃圾回收器是可以在任意时候对其进行回收的，虚引用主要用来跟踪对象被垃圾回收器回收的活动，当被回收时，JVM会把这个弱引用加入到与之相关联的ReferenceQueue中。与软引用和弱引用不同的是，虚引用必须有一个与之关联的ReferenceQueue，通过phantomReference.get()得到的值为null，试想一下，如果没有ReferenceQueue与之关联还有什么存在的价值呢？
```
PhantomReference<Book> phantomReference = new PhantomReference<>(new Book(), referenceQueue);
Book book = phantomReference.get(); //此值为null
Reference reference = referenceQueue.poll();
```

###2、请用Java实现一个线程安全且高效的单例模式。
线程安全且高效的单例模式具体来说有两点，一是懒加载，即需要的时候再去创建该单例，二是由于线程同步带来的性能消耗。

单例模式通常有三个特点：
①构造方法私有化；
②一个静态实例；
③一个全局访问点；

```
public class Singleton {
    private  static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized(Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
这个例子属于“懒汉式”单例模式，通过volatile关键字是修饰静态单例，保证instance在线程之间的可见性，另外有的JVM做了一些优化，导致```instance = new Singleton()```这行代码的指令被重排序，这将会导致有可能instance会先被赋值（即不为null），但是还没有被完全初始化，导致其他线程得到一个不完整的instance，volatile关键字也有阻止指令重排序的功能。

关于线程同步带来的消耗，这里采用double-check模式，只有刚开始的并发线程会进入同步代码块，一旦instance初始化完毕，后面的线程再也不会进入同步代码块，也就没有了线程同步所带来的性能消耗。
