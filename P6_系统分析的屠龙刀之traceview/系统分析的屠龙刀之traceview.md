# P6: 系统分析的屠龙刀之traceview

TraceView是Android平台配备一个很好的性能分析工具，它可以通过图形化的方式让我们了解我们要跟踪的程序的性能，并且能具体到特定的方法，成为了系统开发的另一把屠龙刀。

traceview我主要是用来定位应用短时间的卡，耗时操作。

<img src="..\Images\log_sword.png">


---

[跳转到readme](https://github.com/hfreeman2008/android_core_framework/blob/main/README-CN.md)

---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---

[上一篇文章 P5_系统分析的屠龙刀之dumpsys信息](https://github.com/hfreeman2008/android_core_framework/blob/main/P5_%E7%B3%BB%E7%BB%9F%E5%88%86%E6%9E%90%E7%9A%84%E5%B1%A0%E9%BE%99%E5%88%80%E4%B9%8Bdumpsys%E4%BF%A1%E6%81%AF/%E7%B3%BB%E7%BB%9F%E5%88%86%E6%9E%90%E7%9A%84%E5%B1%A0%E9%BE%99%E5%88%80%E4%B9%8Bdumpsys%E4%BF%A1%E6%81%AF.md)



[下一篇文章 P7_系统分析的屠龙刀之Profile](https://github.com/hfreeman2008/android_core_framework/blob/main/P7_%E7%B3%BB%E7%BB%9F%E5%88%86%E6%9E%90%E7%9A%84%E5%B1%A0%E9%BE%99%E5%88%80%E4%B9%8BProfile/%E7%B3%BB%E7%BB%9F%E5%88%86%E6%9E%90%E7%9A%84%E5%B1%A0%E9%BE%99%E5%88%80%E4%B9%8BProfile.md)


---

# 如何使用
- 通过Android studio自带的traceview查看（Android profiler）


<img src="profiler.png">

通过上面官网说明，DDMS在Android studio 3.0后已经过时了，后面使用Android profiler替换了，所以后详见Android profiler


- 通过Android SDK自带的Debug类

可以在开始的地方调用Debug类的startMethodTracing函数，在要结束的地方调用Debug类的stopMethodTracing函数。

这两个函数运行过程中将采集运行时间内该应用所有线程（注意，只能是Java线程）的函数执行情况，并将采集数据保存到/sdcard/trace_view_debug.trace文件中。

开发者然后需要利用SDK中的Traceview工具来分析这些数据。

代码如下：
```java
//开始的地方
Debug.startMethodTracing("trace_view_debug");  
//结束的地方
Debug.stopMethodTracing();  
```

使用此接口，需要在AndroidManifest.xml文件中定义权限：
```java
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
```

我们将trace_view_debug.trace导出手机：

```java
adb pull /sdcard/trace_view_debug.trace /home/android/
```

分别使用traceview来打开此文件：

如果是Eclipse,则进入/SDK/tools目录，执行如下指令：
```java
./traceview  /home/android/trace_view_debug.trace
```

是Android Studio，则进入android_studio/android-sdk-linux/tools目录，执行指令：
```java
./traceview /home/android/trace_view_debug.trace
```



- 通过DDMS中的traceview查看:

双击自己sdk的monitor.bat
```java
Android\Sdk\tools\monitor.bat
```

---

# 抓取traceview日志

我这以DDMS中的traceview为例，说明如何抓取traceview日志。

(1)双击自己sdk的monitor.bat
```java
Android\Sdk\tools\monitor.bat
```

 
(2)选择自己需要分析问题的应用（如com.kookong.tvplus），点击下图标红的图标1(start method profiling)

<img src="start_method_profiling.png">

点击ok，开始抓取traceview日志

<img src="start_method_profiling_001.png">
 

再点击上面标红的图标1，停止抓取traceview日志。


生成traceview的结果图：


<img src="result.png">

此traceview日志保存的位置一般为：

```java
C:\Users\****\AppData\Local\Temp
```

---

# 打开traceview

我们可以直接打开traceview文件，打开方式为File–Open File：

<img src="Open_File.png">

Traceview界面如下：
<img src="result_detail.png">


Traceview界面包括二个部分，Timeline Panel(时间线面板)和Profile Panel(分析面板)：


Timeline Panel又可细分为左右两个Pane：

- 左边Pane显示的是测试数据中所采集的线程信息：
本次测试数据采集了main线程，ReferenceQueueDaemon，FinallizerDaemo,还有二个Thread的信息。

- 右边Pane所示为时间线，时间线上是每个线程测试时间段内所涉及的函数调用信息:
这些信息包括函数名、函数执行时间等。由上图可知，main线程的工作内容非常多，而其他线程的工作也比较多。



Profile Panel是Traceview的核心界面。

它主要展示了某个线程（先在Timeline Panel中选择线程）中各个函数调用的情况，包括CPU使用时间、调用次数等信息。而这些信息正是查找hotspot的关键依据。

所以，对开发者而言，一定要了解Profile Panel中各列的含义。

Profile Panel各列作用说明如下：
| 列名 | 描述|
|-------|-------|
| Name | 该线程运行过程中所调用的函数名 |
| Incl Cpu Time | 某函数占用的CPU时间，包含内部调用其它函数的CPU时间 |
| Excl Cpu Time | 某函数占用的CPU时间，但不含内部调用其它函数所占用的CPU时间 |
| Incl Real Time | 某函数运行的真实时间（以毫秒为单位），内含调用其它函数所占用的真实时间 |
| Excl Real Time | 某函数运行的真实时间（以毫秒为单位），不含调用其它函数所占用的真实时间 |
| Call+Recur Calls/Total | 某函数被调用次数以及递归调用占总调用次数的百分比 |
| Cpu Time/Call | 某函数调用CPU时间与调用次数的比。相当于该函数平均执行时间 |
| Real Time/Call | 同CPU Time/Call类似，只不过统计单位换成了真实时间 |


另外，每一个Time列还对应有一个用时间百分比来统计的列（如Incl Cpu Time列对应还有一个列名为Incl Cpu Time %的列，表示以时间百分比来统计的Incl Cpu Time）。


---

# 分析traceview
我们重点关注二个hotspot:

- 第一个是调用时间特别长的接口
- 第二种是调用次数特别多的接口

对于第一个是调用时间特别长的接口，我们是以Cpu Time/Call来区分。

我们一般点击这列数据，可以将这个统计数据按倒序排列，也就按方法耗时时间从大到小排列，再过滤我们自己的应用的关键字，定位到耗时方法。

对于第二个是调用次数特别多的接口，我们是以Calls+RecurCalls/Total来区分。
我们一般点击这列数据，可以将这个统计数据按倒序排列，也就按方法的调用次数从大到小排列，再过滤我们自己的应用的关键字，定位到调用次数多的方法。

 
我自己一般的操作是将上面二列拖到最左边，如下图：
<img src="show_result.png">

 
这个图，我们可以看到:
耗时方法名--2是AppWidgetManager$1.onSuccess，占用CPU时间为291ms，调用了1次。

其父调用方法为父调用方法--1，

子被调用方法为子被调用方法--3.

也就是说这个方法的调用顺序为：
父调用方法–1   ========》 耗时方法名–2  ========》 子被调用方法–3



非常的清晰明了。

---

# adb命令抓取apk启动过程的traceview

在应用冷启动过程中，遇到耗时时间长的问题，需要抓取traceview来确认。但是采用monitor的方式抓取，需要点选process，才能进行抓取。在apk未启动时，monitor没有该process的选项，所以无法抓取。

此时使用adb命令抓取apk启动过程的traceview：

```java
adb shell setprop debug.traceview-buffer-size-mb 256
adb shell  am start -n com.android.talkingtom/com.android.talkingtom.Main（apk 启动activity） --start-profiler /data/local/tmp/summer.trace
adb shell am profile stop com.android.talkingtom
adb shell ls -lha /data/local/tmp/
adb pull /data/local/tmp/summer.trace
```


---

# 应用例子

[记一次TextView跑马灯效果导致系统卡的惨案](https://xiaozhuanlan.com/topic/7304691258)

https://xiaozhuanlan.com/topic/7304691258

---

# 参考资料
[android核心技术之性能分析工具TraceView](https://blog.csdn.net/hfreeman2008/article/details/53509489)

https://blog.csdn.net/hfreeman2008/article/details/53509489



---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p6-系统分析的屠龙刀之traceview)

---


[上一篇文章 P5_系统分析的屠龙刀之dumpsys信息](https://github.com/hfreeman2008/android_core_framework/blob/main/P5_%E7%B3%BB%E7%BB%9F%E5%88%86%E6%9E%90%E7%9A%84%E5%B1%A0%E9%BE%99%E5%88%80%E4%B9%8Bdumpsys%E4%BF%A1%E6%81%AF/%E7%B3%BB%E7%BB%9F%E5%88%86%E6%9E%90%E7%9A%84%E5%B1%A0%E9%BE%99%E5%88%80%E4%B9%8Bdumpsys%E4%BF%A1%E6%81%AF.md)



[下一篇文章 P7_系统分析的屠龙刀之Profile](https://github.com/hfreeman2008/android_core_framework/blob/main/P7_%E7%B3%BB%E7%BB%9F%E5%88%86%E6%9E%90%E7%9A%84%E5%B1%A0%E9%BE%99%E5%88%80%E4%B9%8BProfile/%E7%B3%BB%E7%BB%9F%E5%88%86%E6%9E%90%E7%9A%84%E5%B1%A0%E9%BE%99%E5%88%80%E4%B9%8BProfile.md)



---

# 结束语

<img src="../Images/end_001.png">