# 单例模式

>单例模式：确保一个类只有一个实例，并提供一个全局访问点。

单例模式比较出名的两个词：懒汉式、饿汉试，这两个名字比较有意思，“懒”指的是需要的时候才才创建，“饿”则是在类加载的时候直接创建了实例。单例类有一个特点：构造函数私有化，一个静态实例，一个全局访问点。

# 懒汉式（DCL）
```
public class LazySingleton {

    private static volatile LazySingleton instance;

    private LazySingleton() {
    }

    /**
     * 解决同步问题，为了提高性能使用double-check，只有第一次需要同步
     *
     */
    public static LazySingleton getInstance() {
        if (instance == null) {
            synchronized(LazySingleton.class) {
                if (instance == null) {
                    instance = new LazySingleton();
                }
            }
        }
        return instance;
    }
}
```
懒汉式需要注意的一点是同步问题，可以将getInstance()方法改为同步方法，但是这样每次调用getInstance()都会进行同步操作，如果业务场景访问这个方法的并发量比较大，那性能问题就值得考虑，上面的例子采用双重检查加锁的方式来减少使用同步，这样做只有第一次创建的时候会进行同步。其中 volatile 关键字是为了解决instance在线程间的可见性以及防止jvm进行指令重新排序导致得到一个不完整的instance问题。

# 懒汉式-inner class
```
public class Singleton {
	
	private Singleton(){}
	
	public static Singleton getInstance(){
		return SingletonFactory.instance;
	}
	
	private class SingletonFactory(){
		private static Singleton instance = new Singleton();
	}
}

```

# 饿汉试
```
public class EagerlySingleton {

    private static EagerlySingleton instance = new EagerlySingleton();

    private EagerlySingleton() {
    }

    public static EagerlySingleton getInstance() {
        return instance;
    }
}
```
饿汉试没有同步问题（类加载过程是互斥的），但是它会在类加载时就实例化一个对象。


android实际APP 开发中用到的 账号信息对象管理， 数据库对象（SQLiteOpenHelper）等都会用到单例模式

# Android中常见的单例场景
##### 1.LocalBroadcastManager
```
private static final Object mLock = new Object();
private static LocalBroadcastManager mInstance;

public static LocalBroadcastManager getInstance(Context context) {
    synchronized (mLock) {
        if (mInstance == null) {
            mInstance = new LocalBroadcastManager(context.getApplicationContext());
        }
        return mInstance;
    }
}
```    
