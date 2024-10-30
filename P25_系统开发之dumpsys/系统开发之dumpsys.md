# P25: 系统开发之dumpsys

<img src="../flower/flower_p25.png">

---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章结尾</font>](#结束语)

---

# 前言

dumpsys命令是我们系统开发常用的命令之一，对于我们定位系统的具体细节信息是非常有用的；

这篇文章我们深度分析一下dumpsys命令的具体实现；

---

# dumpsys命令

## (1).dumpsys help命令：

```java
adb shell dumpsys --help
usage: dumpsys
         To dump all services.
or:
       dumpsys [-t TIMEOUT] [--priority LEVEL] [--pid] [--thread] [--help | -l | --skip SERVICES | SERVICE [ARGS]]
         --help: shows this help
         -l: only list services, do not dump them
         -t TIMEOUT_SEC: TIMEOUT to use in seconds instead of default 10 seconds
         -T TIMEOUT_MS: TIMEOUT to use in milliseconds instead of default 10 seconds
         --pid: dump PID instead of usual dump
         --thread: dump thread usage instead of usual dump
         --proto: filter services that support dumping data in proto format. Dumps
               will be in proto format.
         --priority LEVEL: filter services based on specified priority
               LEVEL must be one of CRITICAL | HIGH | NORMAL
         --skip SERVICES: dumps all services but SERVICES (comma-separated list)
         SERVICE [ARGS]: dumps only service SERVICE, optionally passing ARGS to it
```


---

## (2).常用dumpsys命令：

```sh
adb shell dumpsys       输出设备中dumpsys信息
adb shell dumpsys -l    输出设备中所有的服务名称

```


```sh
adb shell dumpsys activity -h
Activity manager dump options:
  [-a] [-c] [-p PACKAGE] [-h] [WHAT] ...
  WHAT may be one of:
    a[ctivities]: activity stack state      //activity堆栈状态
    r[recents]: recent activities state     //最近activity的状态
    b[roadcasts] [PACKAGE_NAME] [history [-s]]: broadcast state  //查看广播接收器的信息
    broadcast-stats [PACKAGE_NAME]: aggregated broadcast statistics
    i[ntents] [PACKAGE_NAME]: pending intent state  //挂起的intent状态
    p[rocesses] [PACKAGE_NAME]: process state
    o[om]: out of memory management
    perm[issions]: URI permission grant state
    prov[iders] [COMP_SPEC ...]: content provider state
    provider [COMP_SPEC]: provider client-side state
    s[ervices] [COMP_SPEC ...]: service state
    allowed-associations: current package association restrictions
    as[sociations]: tracked app associations
    exit-info [PACKAGE_NAME]: historical process exit information
    lmk: stats on low memory killer
    lru: raw LRU process list
    binder-proxies: stats on binder objects and IPCs
    settings: currently applied config settings
    service [COMP_SPEC]: service client-side state
    package [PACKAGE_NAME]: all state related to given package
    all: dump all activities
    top: dump the top activity
  WHAT may also be a COMP_SPEC to dump activities.
  COMP_SPEC may be a component name (com.foo/.myApp),
    a partial substring in a component name, a
    hex object identifier.
  -a: include all available server state.
  -c: include client state.
  -p: limit output to given package.
  --checkin: output checkin format, resetting data.
  --C: output checkin format, not resetting data.
  --proto: output dump in protocol buffer format.
  --autofill: dump just the autofill-related state of an activity

```





---

---











---












---




```java


```

---




```java



```



```java



```




```java



```



---

# 参考资料




---

[<font face='黑体' color=#ff0000 size=40 >跳转到文章开始</font>](#p25-系统开发之dumpsys)

---

# 结束语

<img src="../Images/end_001.png">
