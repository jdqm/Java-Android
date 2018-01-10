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