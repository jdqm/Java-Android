ActivityManagerService

AMS掌握了所有应用程序的创建、管理，所以他在framworks层有着举足轻重的作用。AMS的主要功能就是通过Binder进程间通信与Client进程的ActivityManager通信。在创建Activity之后将其交由ActivityManager来管理。它的主要功能有以下3点：

- 四大组件的统一调度

ActivityManagerService最主要的功能就是统一的管理者activity,service,broadcast,provider的创建,运行,关闭
- 进程管理

存储系统所有进程的信息。

- 内存管理

真正的内存管理其实是Linux内核中的内存管理模块和OOM进程一起管理的，Android进程在运行过程中，AMS会把每个应用的oom_adj告诉OOM进程，-17~15，值越小就越重要，回收的优先级的就低。

启动
AMS的启动是在systemserver进程的startBootstrapServices方法中启动的，与PMS类似，但早与PMS。


Activity的启动流程
```
Context#startActivity(Intent);
```

1. ContextWrapper 通过桥接模式，桥接到ContextImpl
2. ContextImpl通过调用Intrumentation#execStartActivity()，在其内部调用ActivityManagerNative.getDefault()拿到AMS在客户端的代理对象，进而将逻辑转移到AMS中；
3. AMS#startActivity()--->ActivityStackSupervisor#startActivityMayWait();
4. 经过一系列的调用，最终调用到ActivityStackSupervisor#realStartActivityLocked()中，在这个方法中通过
app.thread.scheduleLaunchActivity()，将逻辑切换到目标进程的ActivityThread中，具体是通过ActivityThread的内部类ApplicationThread来交互。
5.ApplicationThread#scheduleLaunchActivity()，通过ActivityThread中的 mH发送消息来处理，处理逻辑在H这个Handler中；

6. handleLaunchActivity()-->performLaunchActivity()，创建Activity实例，创建Application（如果之前没有创建），activity.attach(),创建Window(PhoneWindow)。

7. mInstrumentation.callActivityOnCreate(activity, r.state)，activity的onCreate被回调。接着onStart也被回调。

8. 接下来就是调用activity的onResume()，注意前一个Activity onPause后才启动的Activity才能onResume，所以前一个onPause不要做耗时操作，否则后一个Activity启动会变慢。


