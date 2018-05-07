Android的适配问题有很多，主要分为如下几类

#### 1.UI适配
参考文章：http://www.jianshu.com/p/ec5a1a30694b
某些特殊时候，需要根据手机屏幕的宽高来动态地计算布局的大小，另外做界面的时候，用Android Studio自带的预览工具，里面可以选择不同的分辨率，多看看效果就好。

#### 2.API适配
一般都是使用了高版本的API，解决方式是做个判断，高版本使用新API，低版本使用老API即可。

#### 3.字体适配
Android推荐sp来做字体的适配，默认情况下sp与dp是相等的，但sp会随着系统的缩放倍数改变，而dp不会。

###4.drawble与mipmap

xhdpi：2.0
hdpi：1.5
mdpi：1.0（基准）（160*480）
ldpi：0.75


sp： scale Pixel 缩放像素，通常与dp是等值
dp/dip： density independent pixel 设备密度独立像素
dpi： dots per inch 每英寸像素数量
分辨率：屏幕两边的像素数量，例如高位1080个像素，宽有1920个像素，分辨率为1080*1920
屏幕尺寸： 屏幕对角线的长度，例如5寸，4.7寸，当年经典的iPhone4是3.5寸

dp = density * pixel
dpi = 对角线像素数量/对角线长度（屏幕尺寸）