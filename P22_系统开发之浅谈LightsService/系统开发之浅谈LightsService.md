# P21: 系统开发之浅谈LightsService

<img src="../flower/flower_p22.png">



---

# light整体分层架构


![light_whole](light_whole.png)


# LightsService 作用

LightsService 灯光服务提供了LCD背光灯、键盘灯、按键灯、警示灯、电池灯、消息通知灯、蓝牙灯、wifi灯等灯光；

常用到的地方为：PowerManager、NotificationManager、BatteryService等


---

# 获取 LightsService 的方式

```java
方式1
LightsManager manager = (LightsManager) mContext.getSystemService(Context.LIGHTS_SERVICE);

方式2
private static ILightsService sService;
IBinder b = ServiceManager.getService(Context.LIGHTS_SERVICE);
sService = ILightsService.Stub.asInterface(b);

方式3 (system server进程使用)
LightsManager lightsManager = getLocalService(LightsManager.class);
LightsManager lights = LocalServices.getService(LightsManager.class);
```

---

# LightsService 调用流程

![LightsService_call](LightsService_call.png)


---




---

# 日志开关

```java

```


---

# dump信息

```java
adb shell dumpsys input_method
```


---

# handler和消息

```java

```


---

# publishBinderService

onStart()方法中：
```java

```

其他进程获取 InputMethodManagerService :
```java

```

---

# LocalServices--InputMethodManagerInternal


```java

```


在system server进程中：
```java

```

---

# ime命令

```java

```

```java

```

---


# 相关Settings字段


```java

```

```shell

```

---




---



---




---

# 结束语

<img src="../Images/end_001.png">

