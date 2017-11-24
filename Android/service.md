清晰地理解Service。
1、Service的start和bind状态有什么区别？

| 不同点 | start | bind |
| ---- | ---- | ---- |
| 生命周期回调方法不同 |onCreate()->onCommandStart() | onCreate()->onBind() |
|与Context关联性不同|与启动它的Context没有直接关联，即使该Context被销毁了，服务仍可处于启动状态|服务的生命周期与Context相关，Context被销毁了，服务也将停止|
|交互方式不同|启动服务的Context与服务没有直接关联，需要通过其他途径来与服务通信，例如广播|绑定服务的的Context可持有onBind返回的Binder对象，或者其代理对象（跨进程时），可通过它来与服务进行交互|
|停止服务方式不同|在服务内部调用stopSelf()、或者外部stopService()来停止|通过unBindService()来停止|

2、同一个Service，先startService，然后再bindService，如何把它停止掉？
答：需要分别调用stopService（当然也可以是stopSelf）和unbindService才能停止掉。前者是让服务的启动状态失效，后者是让服务的绑定状态清除，调用顺序没有要求。当一个服务既不处于启动状态也没有Context与其绑定时，即可停止掉，

3、你有注意到Service的onStartCommand方法的返回值吗？不同返回值有什么区别？
答：返回值只能是以下4个中的一个，其他的返回值会抛出异常。

|返回值|不同点|
|---|---|
|START_NOT_STICKY|系统在 onStartCommand() 返回后终止服务，则除非有挂起 Intent 要传递，否则系统不会重建服务。这是最安全的选项，可以避免在不必要时以及应用能够轻松重启所有未完成的作业时运行服务。|
|START_STICKY|系统在 onStartCommand() 返回后终止服务，则会重建服务并调用 onStartCommand()，但不会重新传递最后一个 Intent。除非有挂起 Intent 要启动服务（在这种情况下，将传递这些 Intent ），否则系统会通过空 Intent 调用 onStartCommand()。媒体播放器可采用这个返回值。
|START_REDELIVER_INTENT|系统在 onStartCommand() 返回后终止服务，则会重建服务，并通过传递给服务的最后一个 Intent 调用 onStartCommand()。任何挂起 Intent 均依次传递。例如下载服务可以采用这个值。
|START_STICKY_COMPATIBILITY|这个值是START_STICKY的兼容版，不确保服务会被重建，它与START_STICKY的差别是：服务对应的ServiceRecord的callStart成员会被置为false|


4、Service的生命周期方法onCreate、onStart、onBind等运行在哪个线程？
答：运行在宿主进行的主线程中。当startService或者bindService时的大概流程为：
(1)ContextImpl利用AMS在客户端的代理对象向AMS发起远程调用；
(2)AMS通过ApplicationThread的Binder方法与Service宿主进程交互，并通过mH这个Handler将逻辑切换到主线程中，接着onCreate在主线程中被调用；
(3)onStart、onBind的调用过程也是类似，只不过这三个方法都是由AMS发起的单独的一次跨进程调用。