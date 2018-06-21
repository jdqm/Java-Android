### PackageManagerService

由SystemServer启动，具体来说是SystemServer的startBootstrapServices()方法启动，调用了PackageManagerService的main()方法，在main方法中，通过调用PackageManagerServer的构造方法完成创建。

```
public static PackageManagerService main(Context context, Installer installer,
            boolean factoryTest, boolean onlyCore) {
        // Self-check for initial settings.
        PackageManagerServiceCompilerMapping.checkProperties();

        PackageManagerService m = new PackageManagerService(context, installer,
                factoryTest, onlyCore);
        m.enableSystemUserPackages();
        ServiceManager.addService("package", m);
        final PackageManagerNative pmn = m.new PackageManagerNative();
        ServiceManager.addService("package_native", pmn);
        return m;
    }
```

### 1.PMS主要功能

1）初始化相关资源

- 初始化一些uid，例如 android.uid.system；
- 构造一个SystemConfig对象，这个对象在初始化的时候，会读取系统的配置文件，获取到系统的基本权限，硬件支持情况；
- Settings对象，这个对象非常重要，主要是存储系统安装的应用的各种信息；

2）对系统的库进行odex优化
系统首次启动是，我们看到的正在优化多少个应用就是这个操作，除了apk，还会对一些jar包进行odex优化。

3）扫描安装包
- canDirLI这个函数首先会列举目录下面的文件，判断文件是否是apk文件，如果是apk文件，就对其进行解析，然后将解析到的应用保存到系统已安装应用的列表中，同时遍历其四大组件，将其添加到相应的数据结构中，使其公有化。比如说静态注册的广播，就是保存在PMS中，当有广播来的时候，AMS会到PMS中查询匹配的广播接收者。

4）根据扫描结果对安装包进行处理

- PMS在进行后续处理中，主要是针对OTA升级导致的系统应用出现删除的情况进行处理，如果发现之前已经安装OK的应用在这次系统启动中没有找到，而且在扫描完data分区之后，依然没有，就确认此应用已经在OTA升级过程中被删除了，这里就会删除应用的安装信息。然后更新应用的权限信息，因为如果应用在OTA升级过程中进行了更新，那么他的权限需要与升级之前的应用的权限保持一致，然后清除应用扫描过程中的临时文件，最后调用writeLPr将本次开机过程中扫描到的应用信息写入到磁盘保存。在应用信息保存之后，初始化一个mInstallerService对象，用于用户安装三方应用。然后进行一次垃圾回收，至此PMS的初始化就走完了。


### apk安装过程

1. 复制APK安装包到data/app目录下，解压并扫描安装包，把dex文件(Dalvik字节码)保存到dalvik-cache目录，并data/data目录下创建对应的应用数据目录。(卸载过程会删除上面的文件)



    