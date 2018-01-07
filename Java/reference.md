从JDK1.2开始，Java中的引用类型分为四种，分别是：①强引用（StrongReference）、②软引用（softRefernce）、③弱引用（WeakReference）、④虚引用（PhantomReference）。

###1.强引用-StrongReference
这种引用是平时开发中最常用的，例如```String strong = new String("Strong Reference")```，当一个实例对象具有强引用时，垃圾回收器不会回收该对象，当内存不足时，宁愿抛出OutOfMemeryError异常也不会通过回收强引用的对象，因为JVM认为强应用的对象是用户正在使用的对象，它无法分辨出到底该回收哪个，强行回收有可能导致系统严重错误。

###2.软引用-SoftReference
如果一个对象只有软引用，那么只有当内存不足时，JVM才会去回收该对象，其他情况不会回收。软引用可以结合ReferenceQueue来是用，当由于系统内存不足，导致软引用的对象被回收了，JVM会把这个软引用加入到与之相关联的ReferenceQueue中。

```
ReferenceQueue referenceQueue = new ReferenceQueue();
SoftReference<Book> softReference = new SoftReference<>(new Book(), referenceQueue);
Reference reference = referenceQueue.poll();
```
当系统内存不足时，触发gc，这个Book就会被回收，reference 将不为null。

###3.弱引用-WeakReference
只有弱引用的对象，当JVM触发gc时，就会回收该对象。与软引用不同的是，不管是否内存不足，弱引用都会被回收。弱引用可以结合ReferenceQueue来是用，当由于系统触发gc，导致软引用的对象被回收了，JVM会把这个弱引用加入到与之相关联的ReferenceQueue中，不过由于垃圾收集器线程的优先级很低，所以弱引用不一定会被很快回收。下面通过一个主动触发gc的例子来验证此结论。

```
 ReferenceQueue referenceQueue = new ReferenceQueue();
 WeakReference<Book> weakReference = new WeakReference(new Book(), referenceQueue);
 System.gc();
 //Runtime.getRuntime().gc();
 Reference reference = referenceQueue.poll();
```
![](/Resource/weak_reference.png)

###4.虚引用-PhantomReference

