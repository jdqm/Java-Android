# 1. 类加载的七个阶段

## 1.1加载
1. 通过一个类的全限定名来获取定义此类的二进制字节流；
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构；
3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据结构的访问入口；前面说大部分对象都是在堆中创建，但是对于HotSpot虚拟机，这个对象存储在方法区。


## 1.2 验证
1. 文件格式验证
2. 元数据验证
3. 字节码验证
4. 符号引用验证

## 1.3 准备
为类变量分配内存（在方法区）并设置类变量的初始值，通常是零值。

## 1.4 解析
虚拟机将常量池中的符号引用替换为直接引用的过程。

## 1.5 初始化
执行类构造器<clinit>()方法的过程。

## 1.6 使用
## 1.7 卸载

# 2. 类加载器

## 2.1 双亲委派模型-Parents delegation Model

![Parents delegation Model](https://upload-images.jianshu.io/upload_images/3631399-b8a2f379473c38fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 调用 findLoadedClass(String) 检查这个类是否已经加载过；
2. 否则调用父加载器的 loadClass 去加载，如果父加载器为null，则使用启动类加载器作为父加载器Bootstrap Classloader；
3. 否则调用自己的 findClass(String) 去加载这个类；

总的来说就是Class文件加载到类加载子系统后，先沿着图中红色虚线的方向自下而上进行委托，再沿着黑色虚线的方向自上而下进行查找，整个过程就是先上后下。
```
protected Class<?> loadClass(String name, boolean resolve)
	throws ClassNotFoundException {
		// First, check if the class has already been loaded
		Class<?> c = findLoadedClass(name);
		if (c == null) {
			try {
				if (parent != null) {
					c = parent.loadClass(name, false);
				} else {
					c = findBootstrapClassOrNull(name);
				}
			} catch (ClassNotFoundException e) {
				// ClassNotFoundException thrown if class not found
				// from the non-null parent class loader
			}

			if (c == null) {
				// If still not found, then invoke findClass in order
				// to find the class.
				c = findClass(name);
			}
		}
		return c;
}
```

## 2.2 自定义ClassLoader

系统提供的类加载器只能够加载指定目录下的jar包和Class文件，如果想要加载网络上的或者是D盘某一文件中的jar包和Class文件则需要自定义ClassLoader。
实现自定义ClassLoader需要两个步骤：

1.  定义一个自定义ClassLoade并继承抽象类ClassLoader。
2.  复写findClass方法，并在findClass方法中调用defineClass方法。

下面我们就自定义一个ClassLoader用来加载位于D:\lib的Class文件。
### 2.2.1 编写测试Class文件
首先编写测试类并生成Class文件，如下所示。
```
package com.example;
public class Jobs {
    public void say() {
        System.out.println("One more thing");
    }
}
```

将这个Jobs.java放入到D:\lib中，使用cmd命令进入D:\lib目录中，执行Javac Jobs.java对该java文件进行编译，这时会在D:\lib中生成Jobs.class。

## 2.2.2 编写自定义ClassLoader
接下来编写自定义ClassLoader，如下所示。
```
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
public class DiskClassLoader extends ClassLoader {
    private String path;
    public DiskClassLoader(String path) {
        this.path = path;
    }
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        Class clazz = null;
        byte[] classData = loadClassData(name);//1
        if (classData == null) {
            throw new ClassNotFoundException();
        } else {
            clazz= defineClass(name, classData, 0, classData.length);//2
        }
        return clazz;
    }
    private byte[] loadClassData(String name) {
        String fileName = getFileName(name);
        File file = new File(path,fileName);
        InputStream in=null;
        ByteArrayOutputStream out=null;
        try {
             in = new FileInputStream(file);
             out = new ByteArrayOutputStream();
            byte[] buffer = new byte[1024];
            int length=0;
            while ((length = in.read(buffer)) != -1) {
                out.write(buffer, 0, length);
            }
            return out.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                if(in!=null) {
                    in.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try{
                if(out!=null) {
                    out.close();
                }
            }catch (IOException e){
                e.printStackTrace();
            }
        }
        return null;
    }
    private String getFileName(String name) {
        int index = name.lastIndexOf('.');
        if(index == -1){//如果没有找到'.'则直接在末尾添加.class
            return name+".class";
        }else{
            return name.substring(index+1)+".class";
        }
    }
}
```

这段代码有几点需要注意的，注释1处的loadClassData方法会获得class文件的字节码数组，并在注释2处调用defineClass方法将class文件的字节码数组转为Class类的实例。loadClassData方法中需要对流进行操作，关闭流的操作要放在finally语句块中，并且要对in和out分别采用try语句，如果in和out共同在一个try语句中，那么如果in.close()发生异常，则无法执行 out.close()。

最后我们来验证DiskClassLoader是否可用，代码如下所示。
```
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class ClassLoaderTest {
    public static void main(String[] args) {
        DiskClassLoader diskClassLoader = new DiskClassLoader("D:\\lib");//1
        try {
            Class c = diskClassLoader.loadClass("com.example.Jobs");//2
            if (c != null) {
                try {
                    Object obj = c.newInstance();
                    System.out.println(obj.getClass().getClassLoader());
                    Method method = c.getDeclaredMethod("say", null);
                    method.invoke(obj, null);//3
                } catch (InstantiationException | IllegalAccessException
                        | NoSuchMethodException
                        | SecurityException |
                        IllegalArgumentException |
                        InvocationTargetException e) {
                    e.printStackTrace();
                }
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

注释1出创建DiskClassLoader并传入要加载类的路径，注释2处加载Class文件，需要注意的是，不要在项目工程中存在名为com.example.Jobs的Java文件，否则就不会使用DiskClassLoader来加载，而是AppClassLoader来负责加载，这样我们定义DiskClassLoader就变得毫无意义。接下来在注释3通过反射来调用Jobs的say方法，打印结果如下：
```
com.example.DiskClassLoader@4554617c
One more thing
```
使用了DiskClassLoader来加载Class文件，say方法也正确执行，显然我们的目的达到了。