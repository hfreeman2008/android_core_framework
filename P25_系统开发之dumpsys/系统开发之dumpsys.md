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




---

---











---









```java


```


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
