1.使用WebView的页面是非常耗资源的，可以考虑放到一个独立的进程中，减少OOM发生的记录。四大组件可以在AndroidManifest.xml注册时声明process属性，例如
```
android:process=":webview"
``

2.apk瘦身
分析apk压缩文件，到底是哪部分文件占用的存储空间比较大，可以使用解压工具(7-zip)打开apk文件或者在Android Studio中直接双击apk文件即可看到各种类型的文件所占用的存储空间，通常来说都是资源文件占用的比较多；
(1)第一步是去掉无用的资源文件，在as中：Analyze->Run Inspection by Name，输入unused resources，这种方式删除无用资源文件，因为其实在项目周期过长中，视觉可能会更换一些资源文件，但开发人员有可能就忘记删除以前使用的资源文件了。当然也可以通过gradle的shrinkResources与minifyEnabled参数可以简单快速的在编包的时候自动删除无用资源
(2)第二步就是使用代码、颜色、或者shape，甚至自定义绘制等来代替图片，我们知道一张图片是比较大的，如果我们能使用xml来定义shape替代它，那就可以减少一些存储空间；
(3)使用矢量图，SVG是指可伸缩矢量图形 (Scalable Vector Graphics)；这个是5.0后引入的，低版本需要通过兼容包来兼容.

```
android { 
     defaultConfig { 
           vectorDrawables.useSupportLibrary = true  
      }
  }
```

(4)第三步就是说有些图片可能很难替换了，那可以考虑压缩，比图tinypng等技术手段；
(5)另外对于一些lib，或者classes.dex比较大的话，就不好一概而论了，看看是否能减少一些第三方库的依赖，或者so库是否需要导入多种cup架构（arm/x86...）

3.监控方法执行耗时

 使用 https://github.com/JakeWharton/hugo 打印方法的运行时参数、耗时等信息来更细粒度地分析代码；

4.布局优化

5.绘制优化

6.内存泄漏优化

7.线程优化

8.响应速度优化

9