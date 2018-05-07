内存泄漏发生在堆内存中，指的是已经没有用的对象，仍然被引用，导致GC无法回收。在平常的开发中，应该如何来处理内存泄漏问题？第一，我们应该尽量避免写出有内存泄漏的代码，但这个对开发人员的要求较高。第二，如何利用工具来检测现有的代码是否存在内存泄漏，推荐使用[LeakCanary:https://github.com/square/leakcanary][1]，下面是一个例子，先制造一个内存泄漏的场景，看看LeakCanary是如何展示的：

### 1.LeakCanary

```
public interface TaskListener {}
```
```
ublic class TaskManager {

    //使用volatile处理线程间的可见性，以及防止指令重排序
    private static volatile TaskManager instance;

    private List<TaskListener> taskListeners = new ArrayList<>();

    private TaskManager() {
    }

    /**
     * 使用双重检查避免每次都进入同步代码块，这样做只要instance初始化完毕，后续将不再进去同步
     *
     * @return singleton of TaskManager
     */
    public static TaskManager getInstance() {
        if (instance == null) {
            synchronized (TaskManager.class) {
                if (instance == null) {
                    instance = new TaskManager();
                }
            }
        }
        return instance;
    }

    public void registerTaskListener(TaskListener taskListener) {
        taskListeners.add(taskListener);
    }

    public void unRegisterTaskListener(TaskListener taskListener) {
        taskListeners.remove(taskListener);
    }
}
```
```
public class MainActivity extends AppCompatActivity implements TaskListener{

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //将当前Activity注册，理应在onDestroy中解注册，否则Activity将被此单例引用，导致泄漏
        TaskManager.getInstance().registerTaskListener(this);
    }

    @Override
    protected void onDestroy() {
        //解注册后就不会泄漏
        //TaskManager.getInstance().unRegisterTaskListener(this);
        super.onDestroy();
    }
}
```
打开->关闭这个Activity，然后LeakCanary会发出一个通知（这个过程需要一点时间，因为需要解析dump出来的hprof文件），告诉你这个Activity泄漏了，点开这个通知得到如下的界面：

![](/Resource/leak_canary.png)

Leakcanary打出来的Log如下：
```
D/LeakCanary: * com.jdqm.leakdemo.MainActivity has leaked:
01-04 04:10:34.588 5174-5204/com.jdqm.leakdemo:leakcanary D/LeakCanary: * GC ROOT static com.jdqm.leakdemo.TaskManager.instance
01-04 04:10:34.588 5174-5204/com.jdqm.leakdemo:leakcanary D/LeakCanary: * references com.jdqm.leakdemo.TaskManager.taskListeners
01-04 04:10:34.588 5174-5204/com.jdqm.leakdemo:leakcanary D/LeakCanary: * references java.util.ArrayList.elementData
01-04 04:10:34.588 5174-5204/com.jdqm.leakdemo:leakcanary D/LeakCanary: * references array java.lang.Object[].[0]
01-04 04:10:34.588 5174-5204/com.jdqm.leakdemo:leakcanary D/LeakCanary: * leaks com.jdqm.leakdemo.MainActivity instance
01-04 04:10:34.588 5174-5204/com.jdqm.leakdemo:leakcanary D/LeakCanary: * Retaining: 3.9 kB.
```
不管是界点击通知打开的界面还是Log都很相当详细，把引用的路径都给你指出来了。

在这个例子中，TaskManager.instance静态成员，它的生命周期比Activity长，关闭Activity时，TaskManager.instance中的taskListeners集合保存了该Activity的引用，导致无法回收而泄漏。


### 2.在Android Studio中，底部有一个Android Profiler也可以查看

![Android profiler](/Resource/Android_profiler.png)


这种方式需要自己查看引用信息，通常来说都是查看占用比较大的。



### 非静态内部类引起的内存泄漏
```
public class HandlerLeakActivity extends AppCompatActivity {

    private MyHandler myHandler = new MyHandler();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_handler_leak);
        //发送一个延时10秒的消息，这个时候Message被投递到MessageQueue中，而Message的target引用了Handler，
        //该Handler又是非静态内部类，持有外部类的引用，从而导致Activity无法回收
        myHandler.sendEmptyMessageDelayed(0, 10000);
    }

    class MyHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        //移除消息队列中的对应的消息，释放Handler，从而释放Activity
        //myHandler.removeMessages(0);
    }
}
```

可以将其改为静态内部类，以避免此问题。


```
public class StaticHandlerActivity extends AppCompatActivity {

    private MyHandler myHandler = new MyHandler(this);

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_static_handler);
        //同样的演示10秒，在onDestroy中也没有移除消息，但因为Handler对Activity的引用为弱引用，弱引用的对象是可以被回收的，所以不会导致其泄漏
        myHandler.sendEmptyMessageDelayed(0, 10000);
    }

    static class MyHandler extends Handler {
        private WeakReference<Activity> mActivity;

        public MyHandler(Activity Activity) {
            this.mActivity = new WeakReference<>(Activity);
        }

        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            StaticHandlerActivity activity = (StaticHandlerActivity) mActivity.get();
            if (activity != null) {
                activity.setText();
            }
        }
    }

    public void setText() {
        //
    }
}
```
这种情况下，即使在onDestroy中没有移除消息队列中的消息，也不会造成内存泄漏。



[1]: https://github.com/square/leakcanary