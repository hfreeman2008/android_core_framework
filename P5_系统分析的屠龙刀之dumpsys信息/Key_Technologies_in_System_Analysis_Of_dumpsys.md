# P5: Key Technologies in System Analysis - dumpsys

The most commonly used analysis of Android issues is logs。

But when analyzing system issues, we will find that there are too few logs, and even no information at all.

At this point, dumpsys information, due to its particularly complete and comprehensive nature, became another useful tool  in system development.


<img src="..\Images\log_sword.png">

---

As an Android system development engineer, you can only say that you have started to feel the pulse of the system if you have mastered the dumpsys tool。

<img src="mai_bo.png">

---

<img src="jie_gao.png">

---

# dumpsys

View the complete list of system services：

```java
adb shell dumpsys -l
```

---
# key dumpsys command

## dumpsys meminfo

This command is particularly useful for analyzing device memory.

Viewing device memory information：

```java
adb shell dumpsys meminfo
```

View the memory information of the device about application com.android.pkgname：

```java
adb shell dumpsys meminfo com.android.pkgname
```

---

## dumpsys cpuinfo

This command is particularly useful for analyzing the CPU of the device

View CPU information

```java
adb shell dumpsys cpuinfo
```

---

## dumpsys package

This command is particularly useful for analyzing all application information of the device

View application package information

```java
adb shell dumpsys package
adb shell dumpsys package com.android.pkgname
```

---

## dumpsys activity

For the system, the dumpsys command of AMS is an important instruction for us to view system information

```java
adb shell dumpsys activity
```

The dumpsys activity information is too vast and extensive, mainly including the following aspects:

```java
ACTIVITY MANAGER SETTINGS (dumpsys activity settings) activity_manager_constants:
ACTIVITY MANAGER ALLOWED ASSOCIATION STATE (dumpsys activity allowed-associations)
ACTIVITY MANAGER PENDING INTENTS (dumpsys activity intents)//intent
ACTIVITY MANAGER BROADCAST STATE (dumpsys activity broadcasts)//broadcasts
ACTIVITY MANAGER CONTENT PROVIDERS (dumpsys activity providers)//provider
ACTIVITY MANAGER URI PERMISSIONS (dumpsys activity permissions)
ACTIVITY MANAGER SERVICES (dumpsys activity services)  //service
ACTIVITY MANAGER RECENT TASKS (dumpsys activity recents)//recent activity
ACTIVITY MANAGER LAST ANR (dumpsys activity lastanr)//anr
ACTIVITY MANAGER STARTER (dumpsys activity starter)
ACTIVITY MANAGER CONTAINERS (dumpsys activity containers)//activity containers
ACTIVITY MANAGER ACTIVITIES (dumpsys activity activities)//activities
ACTIVITY MANAGER PROCESS EXIT INFO (dumpsys activity exit-info)
ACTIVITY MANAGER LMK KILLS (dumpsys activity lmk)
ACTIVITY MANAGER RUNNING PROCESSES (dumpsys activity processes)
ACTIVITY MANAGER USERS (dumpsys activity users)
Raw LRU list (dumpsys activity lru)
```


the most commonly used one is to view the focus activity：

```java
adb shell "dumpsys activity | grep -A 45 -i 'from top to'"
adb shell "dumpsys | grep -i -A 4 'mCurrentFocus'"
```

---

## dumpsys window

For the system, the dumpsys command of WMS is an important instruction for us to view system information

```java
adb shell dumpsys window
```

dumpsys window mainly including the following aspects：
```java
WINDOW MANAGER LAST ANR (dumpsys window lastanr)
WINDOW MANAGER POLICY STATE (dumpsys window policy)//policy
WINDOW MANAGER ANIMATOR STATE (dumpsys window animator)//animator
WINDOW MANAGER SESSIONS (dumpsys window sessions)
WINDOW MANAGER DISPLAY CONTENTS (dumpsys window displays)
WINDOW MANAGER TOKENS (dumpsys window tokens)
WINDOW MANAGER WINDOWS (dumpsys window windows)//windows
WINDOW MANAGER TRACE (dumpsys window trace)
WINDOW MANAGER LOGGING (dumpsys window logging)
WINDOW MANAGER HIGH REFRESH RATE BLACKLIST (dumpsys window refresh)
WINDOW MANAGER CONSTANTS (dumpsys window constants)
```

view device resolution:

```java
adb shell "dumpsys window | grep init"
```

---

## dumpsys settings

reading data from the SettingsProvider database

```java
adb shell dumpsys settings
```

This command is a commonly used for us to confirm the fields in the SettingsProvider database during development。


---

## dumpsys input_method

View input method information：

```java
adb shell dumpsys input_method
```

Some key information about input methods：

```java
Current Input Method Manager state:
  Input Methods: mMethodMapUpdateCount=2
  InputMethod #0:
    mId=com.google.android.inputmethod.latin/com.android.inputmethod.latin.LatinIME mSettingsActivityName=com.google.android.apps.inputmethod.latin.preference.SettingsActivity 
......

  InputMethod #1:
    mId=com.inputmethod.tvz/com.android.inputmethod.pinyin.TvzIME mSettingsActivityName=com.android.inputmethod.pinyin.SettingsActivity mIsVrOnly=false 
 ......
  mCurMethodId=com.google.android.inputmethod.latin/com.android.inputmethod.latin.LatinIME
```



---

## dumpsys battery

View battery information

```java
adb shell dumpsys battery
adb shell dumpsys batterystats
```


adb shell dumpsys battery:
```java
Current Battery Service state:
  AC powered: false
  USB powered: true
  Wireless powered: false
  Max charging current: 500000
  Max charging voltage: 5000000
  Charge counter: 3270000
  status: 5
  health: 2
  present: true
  level: 100
  scale: 100
  voltage: 4352
  temperature: 274
  technology: Li-ion

```

---

## dumpsys display

View display information：

```java
adb shell dumpsys display
```

View brightness values :

mBrightness：
```java
BrightnessObserver
    mBrightness: 102
```



Information related to automatic brightness adjustment：

```java
SimpleMappingStrategy
  mSpline=MonotoneCubicSpline{[(0.0, 0.011453409: 0.0019675908), (10.0, 0.031129315: 0.0022426946), (30.0, 0.08148529: 0.0028148363), (60.0, 0.17484152: 0.0034001395), (100.0, 0.3223777: 0.00396333), (150.0, 0.53429043: 0.0036997495), (210.0, 0.7239651: 0.0033023162), (212.0175, 0.73091215: 0.003443372), (255.0, 0.8789162: 0.0034433564)]}
  mMaxGamma=3.0
  mAutoBrightnessAdjustment=-0.2651215
  mUserLux=212.0175
  mUserBrightness=0.73091215
```

---

# dumpsys命令的方法实现

我以一个比较简单的系统服务DiskStatsService为例，说明其实是系统服务实现Binder接口中的方法dump。

<img src="dumpsys_implement.png">


IBinder类：
```java
public interface IBinder {
    /**
     * Print the object's state into the given stream.
     * 
     * @param fd The raw file descriptor that the dump is being sent to.
     * @param args additional arguments to the dump request.
     */
    public void dump(@NonNull FileDescriptor fd, @Nullable String[] args) throws RemoteException;

    /**
     * Like {@link #dump(FileDescriptor, String[])} but always executes
     * asynchronously.  If the object is local, a new thread is created
     * to perform the dump.
     *
     * @param fd The raw file descriptor that the dump is being sent to.
     * @param args additional arguments to the dump request.
     */
    public void dumpAsync(@NonNull FileDescriptor fd, @Nullable String[] args)
            throws RemoteException;
}
```



Binder类：

```java
public class Binder implements IBinder {

    /**
     * Print the object's state into the given stream.
     *
     * @param fd The raw file descriptor that the dump is being sent to.
     * @param fout The file to which you should dump your state.  This will be
     * closed for you after you return.
     * @param args additional arguments to the dump request.
     */
    protected void dump(@NonNull FileDescriptor fd, @NonNull PrintWriter fout,
            @Nullable String[] args) {
    }

    /**
     * Implemented to call the more convenient version
     * {@link #dump(FileDescriptor, PrintWriter, String[])}.
     */
    public void dump(@NonNull FileDescriptor fd, @Nullable String[] args) {
        FileOutputStream fout = new FileOutputStream(fd);
        PrintWriter pw = new FastPrintWriter(fout);
        try {
            doDump(fd, pw, args);
        } finally {
            pw.flush();
        }
    }

        /**
     * Like {@link #dump(FileDescriptor, String[])}, but ensures the target
     * executes asynchronously.
     */
    public void dumpAsync(@NonNull final FileDescriptor fd, @Nullable final String[] args) {
        final FileOutputStream fout = new FileOutputStream(fd);
        final PrintWriter pw = new FastPrintWriter(fout);
        Thread thr = new Thread("Binder.dumpAsync") {
            public void run() {
                try {
                    dump(fd, pw, args);
                } finally {
                    pw.flush();
                }
            }
        };
        thr.start();
    }
}
```

DiskStatsService类：

```java
public class DiskStatsService extends Binder {


    @Override
    protected void dump(FileDescriptor fd, PrintWriter pw, String[] args) {
        ......
    }
}
```

---

# 参考资料
[dumpsys](https://developer.android.google.cn/studio/command-line/dumpsys)

https://developer.android.google.cn/studio/command-line/dumpsys


---

# 结束语

<img src="../Images/end_001.png">