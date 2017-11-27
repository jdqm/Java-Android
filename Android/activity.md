#1.Activity的优先级

- onResume状态的Activity优先级最高，即正在操作的Activity；
- 可见的Activity，比如弹出了一个对话框，或者一个非全屏、或者半透明的Activity启动了，但是后面的Activity还是看得见的；
- 处于后台的Activity，即处于stop状态不可见的Activity。

当系统资源不足时，就会按照上面的优先级，优先回收优先级较低的Activity，以释放资源供系统使用。

##2.Activity的启动模式有哪几种，分别用于什么场景？

##3、清晰地描述下onNewIntent和onConfigurationChanged这两个生命周期方法的场景？