 清晰地理解Service。
1、Service的start和bind状态有什么区别？

2、同一个Service，先startService，然后再bindService，如何把它停止掉？

3、你有注意到Service的onStartCommand方法的返回值吗？不同返回值有什么区别？

4、Service的生命周期方法onCreate、onStart、onBind等运行在哪个线程？
答：运行在宿主进行的主线程中。当startService或者bindService时的大概流程为：
(1)ContextImpl利用AMS在客户端的代理对象向AMS发起远程调用；
(2)AMS通过ApplicationThread的Binder方法与Service宿主进程交互，并通过mH这个Handler将逻辑切换到主线程中，接着onCreate在主线程中被调用；
(3)onStart、onBind的调用过程也是类似，每一个方法的调用都是由AMS发起的跨进程调用。

