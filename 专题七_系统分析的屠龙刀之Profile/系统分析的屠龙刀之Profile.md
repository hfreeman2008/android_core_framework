# 专题七: 系统分析的屠龙刀之Profile

Android Profile是性能分析工具traceview的升级版本，功能更强大，使用更方便，成为了系统开发的另一把屠龙刀。

<img src="..\Images\log_sword.png">

# Android Profile主要作用
- Cpu 问题分析
- Memory问题分析

Profile主要是针对某个特定应用，监视Cpu和Memory 二个关键指标：

<img src="profile_whole.png">

如上图，我们可以知道，这是一个针对应用com.android.phone，监视Cpu和Memory的一个图。


# CPU
我们选择某个特定应用，点击CPU，就会看到cpu监视有四个选择：
<img src="cpu_whole.png">


- Callstack Sample Recording

Sample Java/Kotlin and native code using simpleperf
可以看出主要追踪的是java/kotlin 和 native 代码


- System Trace Recording

Traces Java/Kotlin and native code at the Android platform level
可以看出，这个是从系统平台角度来追踪java/kotlin 和 native 代码，内容更多更全面
所以在追踪分析系统cpu时，推荐使用

- Java/Kotlin Method Trace Recording

Instruments Java/Kotlin code using Android Runtime,tracking every method call(this incurs high overhead making timing infomation inaccurate).
可以看出，这个更偏重是追踪Java/Kotlin代码，会追踪每个回调的方法，所以会导致高昂的开销，并有统计时间不准确的问题。
所以在app层应用，短时间的追踪cpu时，推荐使用

- Java/Kotlin Method Trace Recording(legacy)

Sample Java/Kotlin code using Android Runtime。
这个是以前的过时工具


## 一个样例

### 在app中添加一个耗时操作

在一个Activity中的onResume()接口中，我们添加一个sleep 7秒的模拟耗时操作

```java
protected void onResume() {
    super.onResume();

    try {
        Thread.sleep(7000);
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }
}
```
### 抓取统计数据
在activity界面息屏后，选择Java/Kotlin Method Trace Recording，点击start，再点亮屏幕，等待activity界面完全显示后，点击stop。
会自动生成统计结果：
<img src="result_show.png">


### 分析数据




# Memory










