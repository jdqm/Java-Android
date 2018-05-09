# I.运行时五大数据区
1. 方法区（Method Area）
2. 虚拟机栈（Java Virtual Machine M Stack）
3. 本地方法区（Native Method Area）
4. 堆（Heap）
5. 程序计数器（Program counter Register）

![JVM运行时数据区](https://upload-images.jianshu.io/upload_images/3631399-18d8486df3230b12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 程序计数器
线程私有，若执行的是java方法，则记录的是当前正在执行的虚拟机字节码指令地址；若执行的是本地方法，这个计数器为空（Undefined）。此内存区域是在Java虚拟机规范中唯一没有指定OutOfMemoryError的区域。

- 虚拟机栈
线程私有，描述的是方法执行的模型，当执行一个方法时会创建一个栈帧（Stack Frame）用于存储局部变量表、操作数栈、动态链接、方法出口等信息，声明周期与线程相同。每一个方法的执行的完整过程，就对应着一个栈帧在虚拟机栈中的进栈出栈。
此区域有两个异常：当栈深度超过虚拟机的规定时，StackOverFlowErrorl；当扩展时无法申请到足够的内存，OutOfMemeryError。

- 本地方法栈
与虚拟机栈的区别是，虚拟机栈是为执行Java方法服务，而本地方法栈是为执行Native方法服务，同样这个区域也会抛出StackOverFlowErrorl、OutOfMemeryError。

- 堆
线程共享，但也有线程私有的分配缓存TLAB（Thread Local Allocation Buffer），几乎所有的对象、数组都在这个内存区域分配，这个区域也称为GC堆（Garbage Collected Head），不是叫垃圾堆。可分为新生代，老年代，再进一步细分可分为Eden、From Survivor、ToSurvivor。当堆中没有足够的内存完成实例分配且无法扩展时，抛出OutOfMemoryError。

- 方法区
线程共享，用于存储虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。当方法区无法满足内存分配需求时，将抛出OutOfMemoryError。
此区域包含常量池、

# II.对象的创建（HotSpot）
通过new关键字创建（或克隆、反序列化）
1. 首先检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并检查这个符号引用所代表的类是否被加载、解析和初始化过，如果没有则必须先完成类的加载过程；
2. 分配内存，类加载完成后，对象的大小已经确定；分配过程需要考虑同步问题，采用CAS加失败重试；或者采用本地线程分配缓存（TLAB），这种方式只有在TLAB用完的时候才需要同步；
3. 内存分配完成后，就进行零值初始化（不包含对象头），如果是采用TLAB那这个过程也可以提前至TLAB的过程；
4. 执行<init>来进行初始化；

## III.对象内存布局
对象头（Header）
实例数据（Instance Data）
对齐填充（Padding）非必须

其中对齐填充不是非必须的，主要是HotSpot VM的自动内存管理系统要求对象的起始地址必须是8字节的整数倍。

# IV.对象的访问
方式一：采用对象句柄，通过 Java虚拟机栈本地变量表->对象句柄（在堆中划分一个句柄池）->实例数据或者对象类型数据；这种方式在实例对象地址改变时（比如GC后整理内存空间），栈中引用的句柄的地址不需要改变；
![通过句柄访问](https://upload-images.jianshu.io/upload_images/3631399-e65d2642c51dd370.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


方式二：直接引用，栈中的引用存储的是对象的地址，这种情况就需要堆中的对象布局必须考虑如何放置对象类型的相关信息，这种方式就少了一次寻址，速度更快。Sun HotSpot虚拟机就是采用这种方式。
![直接引用访问](https://upload-images.jianshu.io/upload_images/3631399-f941e4205fc9f3ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
