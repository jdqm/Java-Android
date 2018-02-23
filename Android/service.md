####1、Service的start和bind状态有什么区别？
答：start启动service时，service有独立的生命周期，不依赖与该组件；多次调用start方法，会重复调用onStartCommand方法，start启动的service需要调用stopService或者stopself来停止service(IntentService或自动调用stopSelf方法)。

多次调用bind方法，只会调用一次onBind方法。bind绑定的Service，service依赖于这些组件，这些组件全部销毁后，service也随之销毁。


####2、同一个Service，先startService，然后再bindService，如何把它停止掉？
答：不论调用多少次startService，只需调用一次stopService或者stopSelf方法；不同的组件调用了n次bindService，就要对应n次unBindService才能停止。因此此问题需要调用一次stopService和n次unBindService才能停止掉。
>注：这里默认是使用不同的组件调用了n次bindService,同一个组件多次bind算一次。

####3、你有注意到Service的onStartCommand方法的返回值吗？不同返回值有什么区别？
答：返回值只能是以下4个中的一个，其他的返回值会抛出异常。

|返回值|行为特点|
|---|---|
|START_NOT_STICKY|系统在 onStartCommand() 返回后终止服务，则除非有挂起 Intent 要传递，否则系统不会重建服务。这是最安全的选项，可以避免在不必要时以及应用能够轻松重启所有未完成的作业时运行服务。|
|START_STICKY|系统在 onStartCommand() 返回后终止服务，则会重建服务并调用 onStartCommand()，但不会重新传递最后一个 Intent。除非有挂起 Intent 要启动服务（在这种情况下，将传递这些 Intent ），否则系统会通过空 Intent 调用 onStartCommand()。媒体播放器可采用这个返回值。
|START_REDELIVER_INTENT|系统在 onStartCommand() 返回后终止服务，则会重建服务，并通过传递给服务的最后一个 Intent 调用 onStartCommand()。任何挂起 Intent 均依次传递。例如下载服务可以采用这个值。
|START_STICKY_COMPATIBILITY|这个值是START_STICKY的兼容版，不确保服务会被重建，它与START_STICKY的差别是：服务对应的ServiceRecord的callStart成员会被置为false|


####4、Service的生命周期方法onCreate、onStart、onBind等运行在哪个线程？
答：运行在宿主进行的主线程中。当startService或者bindService时的大概流程为：
(1)ContextImpl利用AMS在客户端的代理对象向AMS发起远程调用；
(2)AMS通过ApplicationThread的Binder方法与Service宿主进程交互，并通过mH这个Handler将逻辑切换到主线程中，接着onCreate在主线程中被调用；
(3)onStart、onBind的调用过程也是类似，只不过这三个方法都是由AMS发起的单独的一次跨进程调用。
>注：因为Service的周期方法都时默认跑在主线程中，所以如果想要在service中执行耗时操作，就要另开线程或者使用IntentService，否则可能产生ARN异常。