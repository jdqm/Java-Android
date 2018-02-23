Android App可以接收来自系统和其他App的广播消息，也可以向它们发送广播消息，比较类似于“发布-订阅”的设计模式，本文主要介绍广播的类型，如何注册广播，如何发送广播以及使用广播需要注意的一些事儿。

# I. 广播的分类

1. 无序广播
没有顺序的广播，广播的接收方没有严格的顺序可言，不可中断。

2. 有序广播
在注册时可指定优先级，优先级高的广播接收者优先收到广播，优先级以一个整数来标识，数值越大优先级越高。可中断，可再修饰。

3. 粘滞广播
发出的广播会滞留，注册时间可晚于发送时间，其他功能与无序广播相同。

# II. 注册广播
编写一个广播接收者，通常是继承BroadcastReceiver并重写onReceive(Context,Intent)方法。
```
public class MyReceiver extends BroadcastReceiver {
    
    private static final String TAG = "MyReceiver";
    
    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO: do somethings
        // 这个方法运行在主线程下
        String action = intent.getAction();
        Log.d(TAG, "onReceive: " + action);
    }
}
```
编写完广播接收者后，就需要进行注册（订阅），告诉系统这个广播接收者对哪些广播感兴趣。注册的方式有静态注册和动态注册两种：

- 在AndroidManifest.xml文件中声明（静态注册）

```
<receiver android:name=".MyReceiver">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
        <action android:name="jdqm.intent.action.TEST"/>
    </intent-filter>
</receiver>
```

- 通过Java代码注册（动态注册）

```
IntentFilter intentFilter = new IntentFilter();
intentFilter.addAction("android.intent.action.BOOT_COMPLETED");
intentFilter.addAction("jdqm.intent.action.TEST");
registerReceiver(myReceiver, intentFilter);
```
通过动态注册的广播接收者，在宿主（注册时所使用的Context）的生命周期期间都是有效的。当然你也可以在适当的时间调用unregisterReceiver(BroadcastReceiver)来解除注册，这个“适当”取决于具体的业务需求。例如使用Activity的Context在onCreate(Bundle) 中注册的一个广播接收者，可以在onDestory()方法回调时解除注册来防止广播接收者泄漏。原则：不重复注册，不泄露。

以上注册的广播接收者对 android.intent.action.BOOT_COMPLETED 和 jdqm.intent.action.TEST 这两种action的广播感兴趣，后者是自定义的广播，前者是开机完成时由系统发出（通常自启动的应用会注册这个广播），但注册这个广播须要以下权限:
```
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
```

# III. 发送广播
 
1. 发送无序广播
```
Intent intent = new Intent("jdqm.intent.action.TEST");
sendBroadcast(intent);
```
2. 发送有序广播
```
Intent intent = new Intent("jdqm.intent.action.TEST");
//第二个参数是权限
sendOrderedBroadcast(intent, null);
```
3.  发送本地广播
 
```
Intent intent = new Intent("jdqm.intent.action.TEST");
LocalBroadcastManager.getInstance(this).sendBroadcast(intent);
```
本地广播只有本应用内通过LocalBroadcastManager.getInstance(this).registerReceiver方法注册的广播接收者能收到，具有更高的安全性，效率也更高（不用跨进程通信）。

4. 发送粘滞广播
```
Intent intent = new Intent("jdqm.intent.action.TEST");
sendStickyBroadcast(intent);
```
这种类型广播在Android6.0中已经被标记被过时， 它有不安全(任何App都能访问), 没有保护 (任何App都能修改)等问题。另外发送这种广播需要以下权限
```
<uses-permission android:name="android.permission.BROADCAST_STICKY"/>
```

# IV. 接收顺序

1. 对于无序广播，动态注册的广播接收者会先收到，可以从源码中得到理论支撑。在BroadcastQueue类中有两个集合
```
//存储所有动态注册的无序广播接受者
final ArrayList<BroadcastRecord> mParallelBroadcasts = new ArrayList<>();

//存储所有静态注册和动态注册的有序广播接收者
final ArrayList<BroadcastRecord> mOrderedBroadcasts = new ArrayList<>();
```
然后在其processNextBroadcast(boolean fromMsg)方法中，首先是处理了mParallelBroadcasts集合。
2. 对于有序广播，优先级高的接受者先收到，如果优先级相同，顺序就是不确定的。先收到的接收者可以调用abortBroadcast()来中断此广播，后续优先级较低的接受者将无法收到。除了中断还可以调用setResultXxx()方法来往广播添加数据，后续的接收者可以读取这些数据。

# IV. 安全性与实践

1.  如果你的广播不需要发送给本应用以外的组件，使用LocalBroadcastManager来发送广播，这样安全性和效率都比较高
2. 静态注册有可能造成大量的App启动，这将会影响系统的性能，所以尽量使用动态注册来替代静态注册。这一点Android系统就做出了很好的示范，比如 CONNECTIVITY_ACTION 这个广播只发送给动态注册的广播接收者。 

3. 不在广播的Intent中包含敏感的信息，因为只要注册了这个广播就能读取到这些信息。你可以通过以下3中方式来获得一定的安全性。
-  通过使用权限来发送广播，这样只有声明了该权限的应用才能收到广播。但是你很难确保你的权限不被泄漏。
-  Android4.0及以上版本，在发送广播的时候可以通过setPackage来指定package（可以指定多个），这样只有匹配的package能接受到。
- 使用LocalBroadcastManager来发送本地广播。

4. 当你注册了一个广播，意味着任何App都可以给你发送广播，以下有三点可以限制接收者：
-  注册的时候增加权限。
- 在AndroidManifest.xml注册receivers时，将android:exported属性设为false。
- 使用LocalBroadcastManager来注册。

5. action的命名空间是全局的，这意味着action有可能会与其他App冲突，所以最好是有一个自己的命名空间。

6. 因为广播接收者是运行在主线程，它应该快速地被执行并且return，所以不要在onReveive方法中做比较耗时的操作。
7. 不要在广播接收者中启动activitys，这违背了用户的使用习惯，特别是不止一个接收者时。这种情况下可以考虑展示一个notification来替代。

# V. 其他

1. 系统广播的action的完整列表在Android SDK下的 BROADCAST_ACTIONS.TXT，路径为: Android/sdk/platforms/android-26/data/BROADCAST_ACTIONS.TXT

2. Android3.1开始，系统的package manager将记录处于停止状态的应用。默认情况下，静态注册了广播的处于“停止”状态的应用，是不会被启动的，即不会收到广播。当然你也可以为Intent指定flag来该变这个行为：

- FLAG_INCLUDE_STOPPED_PACKAGES：包含处于“停止”状态的应用；
- FLAG_EXCLUDE_STOPPED_PACKAGES：不包含处于“停止”状态的应用；

如果Intent不包含（或都包含）这两个flag，则表现形式是包含处于“停止”转态的应用，但是系统默认添加了FLAG_EXCLUDE_STOPPED_PACKAGES这个flag，这一点在源码中有所体现：

ActivityManagerService#broadcastIntentLocked

```
final int broadcastIntentLocked(ProcessRecord callerApp,
    String callerPackage, Intent intent, String resolvedType,
    IIntentReceiver resultTo, int resultCode, String resultData,
    Bundle resultExtras, String[] requiredPermissions, int appOp, Bundle bOptions,
    boolean ordered, boolean sticky, int callingPid, int callingUid, int userId) {
    
    intent = new Intent(intent);

    // By default broadcasts do not go to stopped apps.
    intent.addFlags(Intent.FLAG_EXCLUDE_STOPPED_PACKAGES);
    ...
}
```      
这就意味着如果你想启动处于“停止”状态的应用，必须添加FLAG_INCLUDE_STOPPED_PACKAGES这个flag。那么一个应用在什么情况下会处于停止状态？①应用首次安装并且没有启动过；②被人为地强制停止。开机完成的广播就是FLAG_EXCLUDE_STOPPED_PACKAGES这种类型的Intent，这意味着如果你的应用被停止了，开机自启就会失效。[下一篇文章][1]将从源码的角度来分析广播的工作流程。

[1]: http://www.jianshu.com/p/a842432a4fc2