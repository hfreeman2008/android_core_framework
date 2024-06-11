# 专题四: 系统分析的屠龙刀之dumpsys信息
android问题的分析，大家使用最多的是日志。
但是系统问题分析，我们会发现日志太少，甚至是什么信息都没有，这时候dumpsys信息，因其信息特别完整全面，成为了系统开发的另一把屠龙刀。

<img src=".\Images\log_sword.png">

# dumpsys 命令

查看dump系统服务的完整列表：
```java
adb shell dumpsys -l
```

# 关键的dumpsys信息
## dumpsys meminfo
这个命令，我们针对分析设备的memory，特别有用

查看所有应用的内存信息：
```java
adb shell dumpsys meminfo
```

查看设备对应应用com.android.pkgname的内存信息：
```java
adb shell dumpsys meminfo com.android.pkgname
```

## dumpsys cpuinfo
这个命令，我们针对分析设备的cpu，特别有用
查看cpu信息
```java
adb shell dumpsys cpuinfo
```


## dumpsys package
这个命令，我们针对分析设备的所有应用信息，特别有用

查看应用包信息
```java
adb shell dumpsys package
adb shell dumpsys package com.android.pkgname
```







# 参考资料
1.dumpsys
https://developer.android.google.cn/studio/command-line/dumpsys
