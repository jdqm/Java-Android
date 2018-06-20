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




    