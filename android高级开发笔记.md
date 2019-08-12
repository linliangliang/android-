# 如何对 Android 应用进行性能分析

如果不考虑使用其他第三方性能分析工具的话，我们可以直接使用 ddms 中的工具，其实 ddms 工具已经非常的强大了。
ddms 中有 traceview、heap、allocation tracker,Hierarchy Viewer 等工具都可以帮助我们分析应用的方法执行时间效率和内存使用情况。

## TraceView 简介

函数的耗时：这是androidsdk自带的工具，用于测量函数耗时的。
开发者在一些关键代码段开始前调用 Android SDK 中 Debug 类的startMethodTracing 函数，并在关键代码段结束前调用 stopMethodTracing 函数。 这两个函数运行过程中将采集运行时间内该应用所有线程（注意，只能是 Java 线程）的函数执行情况，并将采集数据保存到/mnt/sdcard/下的一个文件中。开发者然后需要利用 SDK 中的 Traceview 工具来分析这些数据。 

## heap 简介
heap 工具可以帮助我们检查代码中是否存在会造成内存泄漏的地方。

用 heap 监测应用进程使用内存情况的步骤如下： 
1．启动 eclipse 后，切换到 DDMS 透视图，并确认 Devices 视图、Heap 视图都是打开的； 
2．点击选中想要监测的进程，比如 system_process 进程； 
3．点击选中 Devices 视图界面中最上方一排图标中的“Update Heap”图标； 
4．点击 Heap 视图中的“Cause GC”按钮； 
5．此时在 Heap 视图中就会看到当前选中的进程的内存使用量的详细情况。 
- 说明：

a. 点击“Cause GC”按钮相当于向虚拟机请求了一次 gc 操作；
b. 当内存使用信息第一次显示以后，无须再不断的点击“Cause GC”，Heap 视图界面会定时刷新，在对应用的不断的操作过程中就可以看到内存使用的变化； 
c. 内存使用信息的各项参数根据名称即可知道其意思，在此不再赘述。 

**如何才能知道我们的程序是否有内存泄漏的可能性呢?
这里需要注意一个值：Heap 视图中部有一个 Type 叫做 data object，即数据对象，也就是我们的程序中大量存在的类类型的对象。在 data object 一行中有一列是“Total Size”，其值就是当前进程中所有 Java 数据对象的内存总量，一般情况下，这个值的大小决定了是否会有内存泄漏。可以这样判断： 
1、 不断的操作当前应用，同时注意观察 data object 的 Total Size 值； 
2、 正常情况下 Total Size 值都会稳定在一个有限的范围内，也就是说由于程序中的的代码良好，没有造成对象不被垃圾回收的情况，所以说虽然我们不断的操作会不断的生成很多对象，而在虚机不断的进行 GC 的过程中，这些对象都被回收了，内存占用量会会落到一个稳定的水平**
3、反之如果代码中存在没有释放对象引用的情况，则 data object 的 Total Size 值在每次 GC 后不会有明显的回落，随着操作次数的增多 Total Size 的值会越来越大，直到到达一个上限后导致进程被 kill 掉。 
4、此处以 system_process 进程为例，在我的测试环境中 system_process 进程所占用的内存的data object 的 Total Size 正常情况下会稳定在 2.2~2.8 之间， 而当其值超过 3.55 后进程就会被kill。 
- 总之，使用 DDMS 的 Heap 视图工具可以很方便的确认我们的程序是否存在内存泄漏的可能性。

## allocation tracker 简介
allocation tracker 是内存分配跟踪工具 

## UI布局的分析，Hierarchy Viewer 

可以看到View的布局层次，以及每个View刷新加载的时间。


# 3.如何避免 OOM 异常
减少内存对象的占用
I.ArrayMap/SparseArray代替hashmap

II.避免在android里面使用Enum

III.减少bitmap的内存占用

inSampleSize：缩放比例，在把图片载入内存之前，我们需要先计算出一个合适的缩放比例，避免不必要的大图载入。
decode format：解码格式，选择ARGB_8888/RBG_565/ARGB_4444/ALPHA_8，存在很大差异。
IV.减少资源图片的大小，过大的图片可以考虑分段加载

内存对象的重复利用
大多数对象的复用，都是利用对象池的技术。

I.listview/gridview/recycleview contentview的复用

II.inBitmap 属性对于内存对象的复用ARGB_8888/RBG_565/ARGB_4444/ALPHA_8

这个方法在某些条件下非常有用，比如要加载上千张图片的时候。

III.避免在ondraw方法里面 new对象

IV.StringBuilder 代替+

# Android 中如何捕获未捕获的异常

关键是实现Thread.UncaughtExceptionHandler
然后是在application的oncreate里面注册。
```java
public class CrashHandler implements Thread.UncaughtExceptionHandler {

    private static CrashHandler instance = null;


    public static synchronized CrashHandler getInstance()
    {
        if(instance == null)
        {
            instance = new CrashHandler();
        }
        return instance;
    }

    public void init(Context context)
    {
        Thread.setDefaultUncaughtExceptionHandler(this);
    }

    @Override
    public void uncaughtException(Thread thread, Throwable ex) {
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.append("Thread:");
        stringBuilder.append(thread.toString());
        stringBuilder.append("\t");
        stringBuilder.append(ex);
        TraceLog.i(stringBuilder.toString());
        TraceLog.printCallStatck(ex);
    }
}

CrashHandler
```

# Framework 工作方式及原理，Activity 是如何生成一个 view 的，机制是什么
Framework是android 系统对 linux kernel，lib库等封装，提供WMS，AMS，bind机制，handler-message机制等方式，供app使用。

简单来说framework就是提供app生存的环境。

- 1）Activity在attch方法的时候，会创建一个phonewindow（window的子类）

- 2）onCreate中的setContentView方法，会创建DecorView

- 3）DecorView 的addview方法，会把layout中的布局加载进来。

# 什么是 AIDL 以及如何使用
# Handler 机制
# 事件分发机制
# 子线程中能不能 new handler？为什么
# Android 中的动画有哪几类，它们的特点和区别是什么
视图动画，或者说补间动画。只是视觉上的一个效果，实际view属性没有变化，性能好，但是支持方式少。

属性动画，通过变化属性来达到动画的效果，性能略差，支持点击等事件。android 3.0

帧动画，通过drawable一帧帧画出来。

Gif动画，原理同上，canvas画出来。
# 如何修改 Activity 进入和退出动画
# SurfaceView & View 的区别
view的更新必须在UI thread中进行

surfaceview会单独有一个线程做ui的更新。

surfaceview 支持open GL绘制。
# 第三方登陆
# 第三方支付
# 自定义控件：绘制圆环的实现过程
# Activity的启动过程（不要回答生命周期）

# 理解Activity，View,Window三者关系
https://www.jianshu.com/p/5297e307a688
