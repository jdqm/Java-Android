> 模板方法模式：在一个方法中定义算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。

![template_method.png](http://upload-images.jianshu.io/upload_images/3631399-5d55138bea932342.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>设计原则：别调用我们，我们会调用你。

说的是具体的实现类不要调用模板抽象类里的方法，需要的时候模板方法自然会调用你。

#####Java API中的一些模板方法
- 数组排序：Arrays.sort()
- InputStream 的read()方法由子类实现，而read(byte[] b, int off, int len)
