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

