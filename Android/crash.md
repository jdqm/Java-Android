####利用CrashHandler处理应用程序崩溃问题

当应用出现未捕获异常时，设备通常会弹出一个停止运行的的对话框，当点击确定后，就会关闭出现异常的应用。但是你会发现，现在有的应用出现异常时时闪退，不再弹窗那个不太美观的停止运行Dialog，这点除了在系统上改进以外，我们也可以在应用层接管这个处理流程。

（1）实现一个 UncaughtExceptionHandler 子类，在 uncaughtException(Thread, Throwable) 方法中实现自己的处理逻辑。
```
public class CrashHandler implements Thread.UncaughtExceptionHandler {
    private static final String TAG = "CrashHandler";

    private volatile static CrashHandler instance;

    private CrashHandler() {
    }

    public static CrashHandler getInstance() {
        if (instance == null) {
            synchronized (CrashHandler.class) {
                if (instance == null) {
                    instance = new CrashHandler();
                }
            }
        }
        return instance;
    }

    @Override
    public void uncaughtException(Thread t, Throwable e) {
        //Todo what you want when crash.
        Log.d(TAG, "uncaughtException: ");
        android.os.Process.killProcess(android.os.Process.myPid());
    }
}
```

（2）修改Thread的DefaultUncaughtExceptionHandler，可在Application onCreate()。
```
public class App extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        CrashHandler crashHandler = CrashHandler.getInstance();
        Thread.setDefaultUncaughtExceptionHandler(crashHandler);
    }
}
```
>note: 不要忘了在AndroidManifest.xml中注册Application。

目前有一些大厂都有向外提供一些日志服务系统，例如腾讯的Bugly、MTA等等，利用它们监控线上产品的异常问题，可帮助完善产品。虽然说出去的产品经过了无数次测试，但测试的设备种类数量都是比较有限的，而用户的机器环境是千变万化的，可能不同的设备厂商对系统做了一些改造，所以用户可以说的产品的最大测试资源。