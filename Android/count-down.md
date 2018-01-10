###1.Timer+TimerTask
```
final Timer timer = new Timer();

TimerTask task = new TimerTask() {
    int count = 10;
    @Override
    public void run() {
        Log.d(TAG, "count: " + count);
        count--;
        if (count == 0) {
            timer.cancel();
        }
    }
};
timer.schedule(task, 0, 1000);
```

Timer内部维护了一个 ```TimerImpl extends Thread```，其内部有一个```TimerTask[] timers```，通过```timer.schedule()```系列方法可以向其投递task，具体的实现方式是通过Thread.wait(long)。

###2.Handler#sendMessageDelayed(msg, delayMillis)
这种应该是比较常用，但需要注意非静态内部类容易引起的内存泄漏问题，因为非静态内部类持有外部类（通常是Activity的引用），若在Activity 要销毁时，Handler还被具体的Message持有，就会导致Activity泄漏。

```
private static Handler handler = new Handler() {
    int count = 10;
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
        if (count > 0) {
            Log.d(TAG, "handleMessage: " + count);
            count--;
            handler1.sendEmptyMessageDelayed(0, 1000);
        }
    }
};

handler.sendEmptyMessageDelayed(0, 1000);
```

这种方式其实也是通过Thread.wait()来实现，Looper、MessageQueue、Handler，Handler发送消息实际上就是往MessageQueue插入消息，并唤醒正在阻塞的线程，MeesageQueue的next方法返回消息，Looper调用Handler的dispatchMesaage分发消息，Handler处理消息，分三种情况，msg.callback是否为null（即Handler#post(Runable)所发送的Runnable对象），不为null则条用其run方法，否则继续检查Handler#mCallback是否为null，不为null则调用其mCallback.handleMessage()，若这两个都为null，则调用Handler的handleMessage()方法，这三者只有一个会被调用，可见Handler的handleMessage()优先级是最低的。