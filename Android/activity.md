##### 1.Activity的优先级

* onResume状态的Activity优先级最高，即正在操作的Activity；
* 可见的Activity，比如弹出了一个对话框，或者一个非全屏、或者半透明的Activity启动了，但是后面的Activity还是看得见的；
* 处于后台的Activity，即处于stop状态不可见的Activity。

当系统资源不足时，就会按照上面的优先级，优先回收优先级较低的Activity，以释放资源供系统使用。

##### 2.Activity的启动模式有哪几种，分别用于什么场景？

| 启动模式 | 特点 | 应用场景 |
| --- | --- | --- |
| standard | 标准模式，也是默认的模式，每次启动该Activity都会创建一个新的实例并压入栈顶。 |  |
| singleTop | 栈顶复用模式，当要启动的Activity实例位于栈顶时，不会创建新的实例，而是复用栈顶的实例，但会回调- onNewIntent方法，其他情况和标准模式一致。 |  |
| singleTask | 栈内服用模式，自带clearTop效果，如果目标任务栈中存在改Activity实例，不管是否位于栈顶都会复用，复用时会将它之上的Activity全部出栈以使自己位于栈顶，并调用它们的onDestroy方法。 需要注意的一点是复用的时候并不会更新activity的mIntent，即通过getIntent得到的是第一次启动时的Intent，除非你在onNewIntent中通过setIntent来修改|  |
| singleInstance | 单实例、独立任务栈模式，这种模式下该Activity独占一个任务栈，它永远位于栈顶。 | ①通话界面 |

##### 3.清晰地描述下onNewIntent和onConfigurationChanged这两个生命周期方法的场景？

##### 4.IntentFilter的匹配规则
- action：Intent必须包含action，且与目标组件中的任意一个ation匹配。
- category：Intent中可以不包含category，但是一旦包含，就必须和目标组件IntentFilter中的category中的任意一个相同，否则匹配失败，值得注意的一点是startActivity()、startActivityForResult()都会默认为Intent添加```<category android:name="android.intent.category.DEFAULT"/>```，所以如果你的Activity如果要支持隐式启动，那IntentFilter就必须包含这个category。
- data：data部分分为mimeType和URI两部分，mimeType如：image/png、text/plain等，URI就相对复杂一些：
```
<scheme>://<host>:<port>/[<paht>|<pathPrefix>|<pathPattern>]
```

例如：
```
content://com.example.test:8080/test/info
http://www.baidu.com:80/test/info
```
匹配规则是mimeType和URI都要匹配，其中pathPattern是具有统配符的，例如```image/*```支持所有的图片，image/png是可以和它匹配的。

URI中的scheme默认为content、file，例如：
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
但scheme是其他的就会报错说找不到对应的Activity，例如
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
