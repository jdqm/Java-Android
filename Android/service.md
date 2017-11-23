清晰地理解Service。
1、Service的start和bind状态有什么区别？

2、同一个Service，先startService，然后再bindService，如何把它停止掉？

3、你有注意到Service的onStartCommand方法的返回值吗？不同返回值有什么区别？
```
void serviceDoneExecutingLocked(ServiceRecord r, int type, int startId, int res) {
    ...
    switch (res) {
        case Service.START_STICKY_COMPATIBILITY:
        case Service.START_STICKY: {
            r.stopIfKilled = false;
            break;
        }
        case Service.START_NOT_STICKY: {
            r.findDeliveredStart(startId, true);
            if (r.getLastStartId() == startId) {
                r.stopIfKilled = true;
            }
            break;
        }
        case Service.START_REDELIVER_INTENT: {
            ServiceRecord.StartItem si = r.findDeliveredStart(startId, false);
            if (si != null) {
                si.deliveryCount = 0;
                si.doneExecutingCount++;
                r.stopIfKilled = true;
            }
            break;
        }
        case Service.START_TASK_REMOVED_COMPLETE: {
            r.findDeliveredStart(startId, true);
            break;
        }
        default:
            throw new IllegalArgumentException(
                    "Unknown service start result: " + res);
    }
    if (res == Service.START_STICKY_COMPATIBILITY) {
        r.callStart = false;
    }
    ...
}                
```

4、Service的生命周期方法onCreate、onStart、onBind等运行在哪个线程？
答：运行在宿主进行的主线程中。当startService或者bindService时的大概流程为：
(1)ContextImpl利用AMS在客户端的代理对象向AMS发起远程调用；
(2)AMS通过ApplicationThread的Binder方法与Service宿主进程交互，并通过mH这个Handler将逻辑切换到主线程中，接着onCreate在主线程中被调用；
(3)onStart、onBind的调用过程也是类似，只不过这三个方法都是由AMS发起的单独的一次跨进程调用。