####1、View的getWidth()和getMeasuredWidth()有什么区别吗？

①在View的测量过程中，onMeasure()方法会调用setMeasuredDimension()方法为成员变量mMeasuredWidth、mMeasuredHeight赋值。
```
public static final int MEASURED_SIZE_MASK = 0x00ffffff;

public final int getMeasuredWidth() {
    return mMeasuredWidth & MEASURED_SIZE_MASK;
}
```
通过getMeasuredWidth()的实现可以看到，它的返回值就是```mMeasuredWidth和MEASURED_SIZE_MASK```进行与运算的结果，即是一个小于或等于```MEASURED_SIZE_MASK```的值，超出部分（包括最高2位SpecMode）都会被置零。

②在View的布局过程中，layout()方法会调用``setFrame()``方法为4个成员变量：mLeft、mTop、mRight、mBottom 赋值，这4个成员变量就是View的左上角坐标(mLeft,mTop)，右下角坐标(mRight,mBottom)，而getWidth()的返回值是```mRight - mLeft```，即View在布局阶段得到的宽度，也是View最终的宽度。

**区别**
-  ```getMeasuredWidth() = mMeasuredWidth & MEASURED_SIZE_MASK```，而 ```getWidth() = mRight - mLeft```，在View的默认实现中它们的返回值是相等的。
-  赋值时机不同，mMeasuredWidth在测量阶段中被赋值，而mRight、mLeft则是在布局阶段被赋值。

####2、如何在onCreate中拿到View的宽度和高度？
从第一个问题的分析我们知道，只有View的相应的成员变量被赋值之后，才能拿到View的宽度和高度。而View的测量、布局过程和Activity的生命周期方法并不是同步执行的，所以在onCreate中并不能保证View已经完成了测量或者布局过程，如果没有完成，那拿到的将会是成员变量的默认初始化值0。

**(1)主动调用View的measure()方法**

- **精确```dp/px```时**
```
int measuredWidth = MeasureSpec.makeMeasureSpec(100, View.MeasureSpec.EXACTLY);
int measuredHeight = MeasureSpec.makeMeasureSpec(200, View.MeasureSpec.EXACTLY);
testView.measure(measuredWidth, measuredHeight);
int width = testView.getMeasuredWidth(); //100
int height = testView.getMeasuredHeight(); //200
```
- **```wrap_content```时**
```
int measuredWidth = MeasureSpec.makeMeasureSpec((1<<30)-1, MeasureSpec.AT_MOST);
int measuredHeight = MeasureSpec.makeMeasureSpec((1<<30)-1, MeasureSpec.AT_MOST);
testView.measure(measuredWidth, measuredHeight);
int width = testView.getMeasuredWidth();
int height = testView.getMeasuredHeight(); 
```

>注意：这种情况下如果父布局比当前View的wrap_content小的话，是不准确的，最终大小不能超过父布局。比如：宽度```wrap_contet```是106px，但父View的宽度是100px，最终的大小是100px。

-  **```match_parent```时**
这种情况下不知道父布局的宽高，所以无法提前知道View的宽高。

**(2)另外一种获取宽高的方式就是通过一些回调方法来获取，这些回调方法的调用时机是在测量、布局阶段完成之后回调的。**

举几个例子：

- onWindowFocusChanged
```
@Override
public void onWindowFocusChanged(boolean hasFocus) {
    super.onWindowFocusChanged(hasFocus);
    if (hasFocus) {
        int width = testView.getMeasuredWidth();
        int height = testView.getMeasuredHeight();
    }
}
```

- 采用IdleHandler实现一个onRenderFinished回调
```
Looper.myQueue().addIdleHandler(new MessageQueue.IdleHandler() {
    @Override
    public boolean queueIdle() {
        int width = testView.getMeasuredWidth();
        int height = testView.getMeasuredHeight();
        return false;
    }
});
```
- ViewTreeObserver观察View的状态
```
ViewTreeObserver viewTreeObserver = testView.getViewTreeObserver();
viewTreeObserver.addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
    @Override
    public void onGlobalLayout() {
        int width = testView.getMeasuredWidth();
        int height = testView.getMeasuredHeight();
    }
});
```

#### 3.View.post(Runnable)
1. View.post(Runnable) 内部会自动分两种情况处理，当 View 还没 attachedToWindow 时，会先将这些 Runnable 操作缓存下来；否则就直接通过 mAttachInfo.mHandler 将这些 Runnable 操作 post 到主线程的 MessageQueue 中等待执行。

2. 如果 View.post(Runnable) 的 Runnable 操作被缓存下来了，那么这些操作将会在 dispatchAttachedToWindow() 被回调时，通过 mAttachInfo.mHandler.post() 发送到主线程的 MessageQueue 中等待执行。

3. mAttachInfo 是 ViewRootImpl 的成员变量，在构造函数中初始化，Activity View 树里所有的子 View 中的 mAttachInfo 都是 ViewRootImpl.mAttachInfo 的引用。

4. mAttachInfo.mHandler 也是 ViewRootImpl 中的成员变量，在声明时就初始化了，所以这个 mHandler 绑定的是主线程的 Looper，所以 View.post() 的操作都会发送到主线程中执行，那么也就支持 UI 操作了。

5. dispatchAttachedToWindow() 被调用的时机是在 ViewRootImol 的 performTraversals() 中，该方法会进行 View 树的测量、布局、绘制三大流程的操作。

6. Handler 消息机制通常情况下是一个 Message 执行完后才去取下一个 Message 来执行（异步 Message 还没接触），所以 View.post(Runnable) 中的 Runnable 操作肯定会在 performMeaure() 之后才执行，所以此时可以获取到 View 的宽高。