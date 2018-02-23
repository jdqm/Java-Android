#### 1.Activity的优先级

* onResume状态的Activity优先级最高，即正在操作的Activity；
* 可见的Activity，比如弹出了一个对话框，或者一个非全屏、半透明的Activity启动了，但是后面的Activity还是看得见的，处于onPause状态；
* 处于后台的Activity，即处于stop状态不可见的Activity。

当系统资源不足时，就会按照上面的优先级，优先回收优先级较低的Activity，以释放资源供系统使用。

#### 2.Activity的启动模式有哪几种，分别用于什么场景？

| 启动模式 | 特点 | 应用场景 |
| --- | --- | --- |
| standard-标准模式|系统的默认的模式，每次启动该Activity都会创建一个新的实例并压入栈顶，如果使用的Context是Acitivity以外的Context如ApplicationContext，这个时候是没有任务栈的，需要指定FLAG_ACTIVITY_NEW_TASK这个标记位创建一个新的任务栈| 没有什么特殊需求就是这种模式|
| singleTop-栈顶复用模式|当要启动的Activity实例位于栈顶时，不会创建新的实例，而是复用栈顶的实例，但会回调onNewIntent方法，其他情况和标准模式一致。 | 接收通知启动的内容显示页面，当收到多条新闻推送时，根据传进来的Intent显示不同的内容，不需要创建多个实例 |
| singleTask-栈内服用模式|如果目标任务栈中存在该Activity实例，不管是否位于栈顶都会复用，前台Acitivity要处于栈顶，所以复用时为了让其在栈顶，就要将其上面的Activity全部出栈，所以该模式自带clearTop属性。 需要注意的一点是复用的时候并不会更新activity的mIntent，即通过getIntent得到的是第一次启动时的Intent，除非你在onNewIntent中通过setIntent来修改| 主界面，例如浏览器的主界面，不管启动多少次，只会创建一次，其余情况回调onNewIntent，并且会清空主界面以上的页面 |
| singleInstance-单实例模式 |这种模式下Activity只能单独在一个任务栈，所以它永远位于栈顶。 | 通话界面、闹钟响铃页面|

#### 3.清晰地描述下onNewIntent和onConfigurationChanged这两个生命周期方法的场景？
- onNewIntent：在singleTop、singleTask、singleInstance模式下，若发生重用Activity实例，则会回调onNewIntent，在onNewIntent中可以通过setIntent来刷新getIntent的到数据。
- onConfigurationChanged：在App运行期间，若系统的配置信息发生了改变，例如屏幕方向、语言、键盘参数等信息发生了改变，系统默认的处理方式是销毁当前Activity，但有些情况我们不希望这样做，例如在视频播放时横竖屏切换并不希望重新创建该Activity，这是我们需要在AndroidManifest.xml中<activity>标签中声明android:configChanges属性，例如希望横竖屏切换不销毁重建通常会声明这些属性： ```screenSize|orientation|keyboardHidden。```。这个时候系统会回调onConfigurationChanged来告诉我们系统配置发生了变化，onConfigurationChanged参数中包含一个Configuration对象，可以从中读取最新的配置信息来适配UI。
>**注：从API13开始屏幕旋转 screenSize也会发生改变。**

#### 4.IntentFilter的匹配规则
- action：Intent必须包含action，且与目标组件中的任意一个ation匹配。
- category：Intent中可以不包含category，但是一旦包含，就必须和目标组件IntentFilter中的category中的任意一个相同，否则匹配失败，值得注意的一点是startActivity()、startActivityForResult()都会默认为Intent添加```<category android:name="android.intent.category.DEFAULT"/>```，所以如果你的Activity如果要支持隐式启动，那IntentFilter就必须包含这个category。
- data：data部分分为mimeType和URI两部分，mimeType如：image/png、text/plain等，URI就相对复杂一些：
```
<schema>://<host>:<port>/[<paht>|<pathPrefix>|<pathPattern>]
```

例如：
```
content://com.example.test:8080/test/info
http://www.baidu.com:80/test/info
```
匹配规则是mimeType和URI都要匹配，其中pathPattern是具有统配符的，例如```image/*```支持所有的图片，image/png是可以和它匹配的。

URI中的schema默认为content、file，例如：
```
<intent-filter>
    <action android:name="jdqm.intent.action.SecondActivity"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <data android:mimeType="image/*"/>
</intent-filter>
```
下面这样写是可以匹配的：

```
Intent intent = new Intent();
intent.setAction("jdqm.intent.action.SecondActivity");
intent.setDataAndType(Uri.parse("content://test"), "image/png");
//intent.setDataAndType(Uri.parse("file://test"), "image/png");
startActivity(intent);
```
但schema是其他的就会报错说找不到对应的Activity，例如
```
intent.setDataAndType(Uri.parse("http://test"), "image/png");
```
所以如果使用隐式启动一个Activity，最好是提前检查一下是否有匹配的Activity，没有就不做start逻辑了。
```
PackageManager packageManager = getPackageManager();
//MATCH_DEFAULT_ONLY 是匹配具有category android.intent.category.DEFAULT 的组件
ResolveInfo resolveInfo = packageManager.resolveActivity(intent, PackageManager.MATCH_DEFAULT_ONLY);
if (resolveInfo != null) {
    startActivity(intent);
}

List<ResolveInfo> resolveInfos = packageManager.queryIntentActivities(intent, PackageManager.MATCH_DEFAULT_ONLY);
if (!resolveInfos.isEmpty()) {
    startActivity(intent);
}
```

####5.startActivityForResult 使用场景是什么？ requestCode、 resultCode 含义是什么？
#####5.1 使用场景

用户开始新的活动，并且希望得到新活动的某些信息。比如选择照片、选择联系人、选择收货地址、进行某块数据编辑工作等。
#####5.2 requestCode
- 解决的是「区分多个异步任务」的问题。与其他异步 API 的设计类似，如果没有这个信息，那么 Activity 在收到响应时会进入混乱的状态。比如它不知道自己得到的是选择照片还是选择联系人的结果。
- 该信息会发送到 AMS 那边的 ActivityRecord.requestCode 变量进行记录，Client 端新 Activity 并不知道这个信息。

**为什么 requestCode < 0 时收不到结果？**

ActivityStarter 收到 startActivityLocked 时，写入 ActivityRecord.resultTo 变量为空
```
if (requestCode >= 0 && !sourceRecord.finishing) {
    // 只有非负数时新的 ActivityRecord 对象的 resultTo 变量才指向发起者 ActivityRecord 对象
    resultRecord = sourceRecord;
}
```      
      
在 ActivityStack 收到 finishActivityResultsLocked 时，读取 ActivityRecord.resultTo 变量为空，结果数据不会添加到源 ActivityRecord.results 变量

在 ActivityStack 收到 resumeTopActivityInnerLocked 时，读取 ActivityRecord.results 数组为空，不会分发结果数据，这样源 Activity 也就没有结果回调了

#####5.3 resultCode
- 异步调用结果码，告诉调用者成功/失败/其它信息
- 该信息由被调用 Activity/framework 写入，并经过 AMS 传递给源 Activity

**A 启动 B ，B 中何时执行 setResult， setResult 是否可以位于 finish 之后？** 

setResult 在 finish 之前执行，只是把数据记录在 Activity.mResultCode 和 Activity.mResultData 变量中

最早 Activity 构造器阶段
最晚 Activity.finalize 内存回收阶段
```
// Home 键 + 不保留后台 Activity 可触发 onDestroy
protected void onDestroy() {
    super.onDestroy();
    Log.d(TAG, "onDestroy() called");
    new ReqGC().start();
}

@Override
protected void finalize() throws Throwable {
    Log.d(TAG, "finalize() called");
    finish();
    super.finalize();
}

@Override
public void finish() {
    Log.d(TAG, "finish() called");
    setResult(RESULT_OK, new Intent().putExtra("key", "resultData"));
    super.finish();
}

// 不泄漏 Activity 对象
static class ReqGC {

    public void start() {
        final Handler handler = new Handler();
        handler.postDelayed(new Runnable() {
            @Override
            public void run() {
                System.gc();
                handler.postDelayed(this, 10);
            }
        }, 10);
    }
}
```
如果位于 finish 之后执行，那信息无法传递给源 Activity
从代码可以看出 setResult 和 finish 类似生产者/消费者模型，setResult 负责写入数据，finish 负责读取数据。所以setResult不能位于finish后。

**线程安全问题**

- Activity.mResultCode 和 Activity.mResultData 变量由 Activity 对象的锁进行保护
- 支持后台线程和 UI 线程分别进行 setResult 和 finish
- 但是为什么需要加锁保护这两个信息？需要「解决什么问题」？

**API 设计/数据组装问题**

- 底层 AMS 提供的接口的参数是 setResult 和 finish 的参数的组合形式，但是 Activity 为什么把一个接口拆分成两个接口给开发者使用？

- 使用方便。很多情况下调用者只关心 finish ，不需要理解太多的信息

- API 内部原理/数据处理流程

**关键节点：**
    
- Client 端通过 AMP 把数据发送给 Server 端 AMS Binder 实体
- AMS 把数据包装成 ActivityResult 并保存在源 ActivityRecord 的 results 变量中
- AMS 通过 ApplicationThreadProxy 向 Client 端发送 pause 信息让栈顶 Activity 进入 paused 状态，并等待 Client 端回复或超时
- AMS 接收 Client 端已 paused 信息，恢复下一个获取焦点的 Activity ，读取之前保存在 ActivityRecord.results 变量的数据派发给 Client 端对应的 Activity
- Client 端数据经过 ApplicationThread 对象、ActivityThread 对象的分发最后到达 Activity

**startActivityForResult 和 singleTask 导致源 Activity 收不到正确结果问题**
**基本原则**

源 Activity 和目标 Activity 无法在跨 Task 情况下通过 onActivityResult 传递数据。Android 5.0 以上 AMS 在处理 manifest.xml 文件中的 singleTask 和 singleInstance 信息「不会」创建新的 Task，因此可以收到正常回调

Android 4.4.4 以下 AMS 在处理 manifest.xml 文件中的 singleTask 和 singleInstance 信息「会」创建新的 Task，因此在 startActivity 之后立即收到取消的回调

通过 dumpsys activity activities 命令查看 AMS 状态，验证两个 Activity 是否属于不同的 Task
