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


- Callstack Sample Recording(对 C/C++ 函数采样)

捕获应用的原生线程的采样跟踪数据。在内部，此配置使用 simpleperf 跟踪应用的原生代码。

- System Trace Recording(跟踪系统调用)

捕获非常翔实的细节，以便您检查应用与系统资源的交互情况。
您可以检查线程状态的确切时间和持续时间、直观地查看所有内核的CPU瓶颈在何处，并添加需分析的自定义跟踪事件。


- Java/Kotlin Method Trace Recording

在运行时检测应用，从而在每个方法调用开始和结束时记录一个时间戳。
系统会收集并比较这些时间戳，以生成方法跟踪数据，包括时间信息和 CPU 使用率。
请注意，与检测每个方法相关的开销会影响运行时性能，并且可能会影响分析数据；对于生命周期相对较短的方法，这一点更为明显。
此外，如果应用在短时间内执行大量方法，则分析器可能很快就会超出其文件大小限制，因而不能再记录更多跟踪数据。

- Java/Kotlin Method Trace Recording(legacy)

在应用的 Java 代码执行期间，频繁捕获应用的调用堆栈。分析器会比较捕获的数据集，以推导与应用的 Java 代码执行有关的时间和资源使用信息。
基于采样的跟踪存在一个固有的问题，那就是如果应用在捕获调用堆栈后进入一个方法并在下次捕获前退出该方法，分析器将不会记录该方法调用。
如果您想要跟踪生命周期如此短的方法，应使用插桩跟踪。





# Memory










