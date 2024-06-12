# 专题五: 系统分析的屠龙刀之dumpsys信息
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

## dumpsys activity
对于系统来说，ams的dumpsys命令是我们查看系统信息的重要指令

```java
adb shell dumpsys activity
```

dumpsys activity信息实在是太庞大太多，主要包括如下几个方面：
```java
ACTIVITY MANAGER SETTINGS (dumpsys activity settings) activity_manager_constants:
ACTIVITY MANAGER ALLOWED ASSOCIATION STATE (dumpsys activity allowed-associations)
ACTIVITY MANAGER PENDING INTENTS (dumpsys activity intents)//intent信息
ACTIVITY MANAGER BROADCAST STATE (dumpsys activity broadcasts)//broadcasts信息
ACTIVITY MANAGER CONTENT PROVIDERS (dumpsys activity providers)//provider信息
ACTIVITY MANAGER URI PERMISSIONS (dumpsys activity permissions)
ACTIVITY MANAGER SERVICES (dumpsys activity services)  //服务信息
ACTIVITY MANAGER RECENT TASKS (dumpsys activity recents)//最近应用的信息
ACTIVITY MANAGER LAST ANR (dumpsys activity lastanr)//anr信息
ACTIVITY MANAGER STARTER (dumpsys activity starter)
ACTIVITY MANAGER CONTAINERS (dumpsys activity containers)//activity的容器信息
ACTIVITY MANAGER ACTIVITIES (dumpsys activity activities)//activities信息
ACTIVITY MANAGER PROCESS EXIT INFO (dumpsys activity exit-info)
ACTIVITY MANAGER LMK KILLS (dumpsys activity lmk)
ACTIVITY MANAGER RUNNING PROCESSES (dumpsys activity processes)
ACTIVITY MANAGER USERS (dumpsys activity users)
Raw LRU list (dumpsys activity lru)
```

其中，我们平常用的最多的就是查看当前界面的activity信息：
```java
adb shell "dumpsys activity | grep -A 45 -i 'from top to'"
adb shell "dumpsys | grep -i -A 4 'mCurrentFocus'"
```

## dumpsys window
对于系统来说，wms的dumpsys命令是我们查看系统信息的重要指令

```java
adb shell dumpsys window
```

dumpsys window 主要包括如下几个方面：
```java
WINDOW MANAGER LAST ANR (dumpsys window lastanr)
WINDOW MANAGER POLICY STATE (dumpsys window policy)//策略信息
WINDOW MANAGER ANIMATOR STATE (dumpsys window animator)//窗口动画
WINDOW MANAGER SESSIONS (dumpsys window sessions)
WINDOW MANAGER DISPLAY CONTENTS (dumpsys window displays)
WINDOW MANAGER TOKENS (dumpsys window tokens)
WINDOW MANAGER WINDOWS (dumpsys window windows)//windows信息
WINDOW MANAGER TRACE (dumpsys window trace)
WINDOW MANAGER LOGGING (dumpsys window logging)
WINDOW MANAGER HIGH REFRESH RATE BLACKLIST (dumpsys window refresh)
WINDOW MANAGER CONSTANTS (dumpsys window constants)
```

查看设备分辨率
```java
adb shell "dumpsys window | grep init"
```


## dumpsys settings

读取SettingsProvider数据库中的数据
```java
adb shell dumpsys settings
```

这个命令，是我们开发时确认SettingsProvider数据库的字段的一个常用命令。
















# 参考资料
1.dumpsys
https://developer.android.google.cn/studio/command-line/dumpsys
